<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
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
      <el-form-item label="所属部门" prop="departmentName">
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
        <el-input v-model="dataForm.departmentName" v-popover:deptListPopover :readonly="true" placeholder="点击选择所属部门"
                  class="menu-list__input"></el-input>
      </el-form-item>
      <el-form-item label="职务名称" prop="jobName">
        <el-input v-model="dataForm.jobName" placeholder="职务名称"></el-input>
      </el-form-item>
      <el-form-item label="职务代码" prop="jobCode">
        <el-input v-model="dataForm.jobCode" placeholder="职务代码"></el-input>
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
            enterpriseId: 0,
            departmentId: 0,
            jobCode: '',
            jobName: '',
            parameter1: '',
            parameter2: '',
            departmentName: ''
          },
          unitList: [],
          deptList: [],
          deptListTreeProps: {
            label: 'name',
            children: 'children'
          },
          dataRule: {
            enterpriseId: [
                        {required: true, message: '所属企业不能为空', trigger: 'blur'}
            ],
            departmentName: [
                        {required: true, message: '所属部门不能为空', trigger: 'blur'}
            ],
            jobCode: [
                        {required: true, message: '职务代码不能为空', trigger: 'blur'}
            ],
            jobName: [
                        {required: true, message: '职务名称不能为空', trigger: 'blur'}
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
                url: this.$http.adornUrl(`/enterprise/enterpriseJob/info/${this.dataForm.id}`),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  console.log('enterpriseJob', data)
                  this.dataForm.departmentId = data.data.departmentId
                  this.dataForm.jobCode = data.data.jobCode
                  this.dataForm.jobName = data.data.jobName
                  this.dataForm.parameter1 = data.data.parameter1
                  this.dataForm.parameter2 = data.data.parameter2
                  this.dataForm.enterpriseId = data.data.enterpriseDepartment.enterpriseId
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
                url: this.$http.adornUrl(`/enterprise/enterpriseJob/${!this.dataForm.id ? 'save' : 'update'}`),
                method: 'post',
                data: this.$http.adornData({
                  'id': this.dataForm.id || undefined,
                  'departmentId': this.dataForm.departmentId,
                  'jobCode': this.dataForm.jobCode,
                  'jobName': this.dataForm.jobName,
                  'parameter1': this.dataForm.parameter1,
                  'parameter2': this.dataForm.parameter2,
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
          })
        },
            // 企业部门树选中
        deptListTreeCurrentChangeHandle (data, node) {
          this.dataForm.departmentId = data.id
          this.dataForm.departmentName = data.name
        },
            // 企业部门树设置当前选中节点
        deptListTreeSetCurrentNode () {
          this.$refs.deptListTree.setCurrentKey(this.dataForm.departmentId)
          this.dataForm.departmentName = (this.$refs.deptListTree.getCurrentNode() || {})['name']
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
