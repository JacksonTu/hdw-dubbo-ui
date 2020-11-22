<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="120px">
      <el-form-item label="所属企业" prop="enterpriseId">
        <el-select clearable filterable v-model="dataForm.enterpriseId" placeholder="请选择"
                   @change="handleChangeEnterprise()">
          <el-option
            v-for="item in unitList"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          >
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="所属父部门" prop="parentName">
        <el-popover
          ref="deptListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="deptList"
            :props="deptListTreeProps"
            node-key="id"
            ref="deptListTree"
            @current-change="deptListTreeCurrentChangeHandle"
            :default-expand-all="false"
            :highlight-current="true"
            :expand-on-click-node="false" class="tree">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.parentName" v-popover:deptListPopover :readonly="true" placeholder="点击选择所属部门"
                  class="menu-list__input"></el-input>
      </el-form-item>
      <el-form-item label="部门名称" prop="departmentName">
        <el-input v-model="dataForm.departmentName" placeholder="部门名称"></el-input>
      </el-form-item>
      <el-form-item label="部门代码" prop="departmentCode">
        <el-input v-model="dataForm.departmentCode" placeholder="部门代码(可添加多个部门ID，用逗号隔开，表示该部门可以管理多个部门)"></el-input>
      </el-form-item>
      <el-form-item label="预留1" prop="parameter1">
        <el-input v-model="dataForm.parameter1" placeholder="预留1"></el-input>
      </el-form-item>
      <el-form-item label="预留2" prop="parameter2">
        <el-input v-model="dataForm.parameter2" placeholder="预留2"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
    import {treeDataTranslate} from '@/utils'

    export default {
      data () {
        return {
          visible: false,
          dataForm: {
            id: 0,
            parentId: '',
            enterpriseId: '',
            departmentCode: '',
            departmentName: '',
            parameter1: '',
            parameter2: '',
            parentName: ''
          },
          unitList: [],
          deptList: [],
          deptListTreeProps: {
            label: 'name',
            children: 'children'
          },
          dataRule: {
            parentName: [
                        {required: true, message: '所属父部门不能为空', trigger: 'blur'}
            ],
            enterpriseId: [
                        {required: true, message: '所属企业不能为空', trigger: 'blur'}
            ],
            departmentCode: [
                        {required: true, message: '部门代码不能为空', trigger: 'blur'}
            ],
            departmentName: [
                        {required: true, message: '部门名称不能为空', trigger: 'blur'}
            ],
            parameter1: [
                        {required: false, message: '预留1不能为空', trigger: 'blur'}
            ],
            parameter2: [
                        {required: false, message: '预留2不能为空', trigger: 'blur'}
            ]
          }
        }
      },
      methods: {
        init (id) {
          this.dataForm.id = id || 0
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
            this.unitList = []
            this.deptList = []
            this.getUnitList()
            this.getDeptList()
            if (this.dataForm.id) {
              this.$http({
                url: this.$http.adornUrl(`/enterprise/enterpriseDepartment/info/${this.dataForm.id}`),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.dataForm.parentId = data.data.parentId
                  this.dataForm.enterpriseId = data.data.enterpriseId
                  this.dataForm.departmentCode = data.data.departmentCode
                  this.dataForm.departmentName = data.data.departmentName
                  this.dataForm.parameter1 = data.data.parameter1
                  this.dataForm.parameter2 = data.data.parameter2
                  this.deptListTreeSetCurrentNode()
                }
              })
            }
          })
        },
        // 表单提交
        dataFormSubmit () {
          this.$refs['dataForm'].validate((valid) => {
            if (valid) {
              this.$http({
                url: this.$http.adornUrl(`/enterprise/enterpriseDepartment/${!this.dataForm.id ? 'save' : 'update'}`),
                method: 'post',
                data: this.$http.adornData({
                  'id': this.dataForm.id || undefined,
                  'parentId': this.dataForm.parentId,
                  'enterpriseId': this.dataForm.enterpriseId,
                  'departmentCode': this.dataForm.departmentCode,
                  'departmentName': this.dataForm.departmentName,
                  'parameter1': this.dataForm.parameter1,
                  'parameter2': this.dataForm.parameter2
                })
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.$message({
                    message: '操作成功',
                    type: 'success',
                    duration: 1500,
                    onClose: () => {
                      this.visible = false
                      this.$emit('refreshDataList')
                    }
                  })
                } else {
                  this.$message.error(data.msg)
                }
              })
            }
          })
        },
            // 获取企业树
        getUnitList () {
          this.$http({
            url: this.$http.adornUrl('/enterprise/getEnterpriseTree'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            console.log('getEnterpriseTree', data)
            this.unitList = data.data
          })
        },
            // 获取企业部门树
        getDeptList () {
          this.dataForm.enterpriseId = this.dataForm.enterpriseId || ''
          this.deptList = []
          this.$http({
            url: this.$http.adornUrl('/enterprise/enterpriseDepartment/getDeptSelectTree?enterpriseId=' + this.dataForm.enterpriseId),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.deptList = treeDataTranslate(data.data, 'id', 'parentId')
            console.log('deptList', data)
          })
        },
            // 企业部门树选中
        deptListTreeCurrentChangeHandle (data, node) {
          this.dataForm.parentId = data.id
          this.dataForm.parentName = data.name
        },
            // 企业部门树设置当前选中节点
        deptListTreeSetCurrentNode () {
          this.$refs.deptListTree.setCurrentKey(this.dataForm.parentId)
          this.dataForm.parentName = (this.$refs.deptListTree.getCurrentNode() || {})['name']
        },
            // 企业选择改变
        handleChangeEnterprise () {
          this.getDeptList()
        }
      }
    }
</script>
<style scoped>
 .tree {
    max-height: 250px;
    overflow-y: auto;
  }
  /*滚动条整体部分*/
  .tree::-webkit-scrollbar {
    width: 6px;
    background-color: rgba(217, 217, 217, 0.3);
  }

  /*滚动条里面的小方块，能向上向下移动*/
  .tree::-webkit-scrollbar-thumb {
    background-color: rgba(217, 217, 217, 0.3);
    border-radius: 6px;
  }

  /*滚动条的轨道（里面装有Thumb）*/
  .tree::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px rgba(217, 217, 217, 0.3);
    background-color: #fff;
  }
</style>
