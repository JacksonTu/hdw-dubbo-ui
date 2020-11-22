<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="类型" prop="type">
        <el-radio-group v-model="dataForm.type">
          <el-radio v-for="(type, index) in dataForm.typeList" :label="index" :key="index">{{ type }}</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item :label="dataForm.typeList[dataForm.type] + '名称'" prop="name">
        <el-input v-model="dataForm.name" :placeholder="dataForm.typeList[dataForm.type] + '名称'"></el-input>
      </el-form-item>
      <el-form-item :label="dataForm.typeList[dataForm.type] + '描述'" prop="description">
        <el-input v-model="dataForm.description" :placeholder="dataForm.typeList[dataForm.type] + '描述'"></el-input>
      </el-form-item>
      <el-form-item label="上级菜单" prop="parentName">
        <el-popover
          ref="menuListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="menuList"
            :props="menuListTreeProps"
            node-key="id"
            ref="menuListTree"
            @current-change="menuListTreeCurrentChangeHandle"
            :default-expand-all="false"
            :highlight-current="true"
            :expand-on-click-node="false" class="tree">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.parentName" v-popover:menuListPopover :readonly="true" placeholder="点击选择上级菜单"
                  class="menu-list__input"></el-input>
      </el-form-item>
      <el-form-item v-if="dataForm.type !== 0" label="菜单路由/权限" prop="url">
        <el-input v-model="dataForm.url" placeholder="菜单路由/权限"></el-input>
      </el-form-item>
      <el-form-item label="排序号" prop="seq">
        <el-input-number v-model="dataForm.seq" controls-position="right" :min="0" label="排序号"></el-input-number>
      </el-form-item>
      <el-form-item v-if="dataForm.type !== 2" label="菜单图标" prop="icon">
        <el-row>
          <el-col :span="22">
            <el-popover
              ref="iconListPopover"
              placement="bottom-start"
              trigger="click"
              popper-class="mod-menu__icon-popover">
              <div class="mod-menu__icon-list">
                <el-button
                  v-for="(item, index) in iconList"
                  :key="index"
                  @click="iconActiveHandle(item)"
                  :class="{ 'is-active': item === dataForm.icon }">
                  <icon-svg :name="item"></icon-svg>
                </el-button>
              </div>
            </el-popover>
            <el-input v-model="dataForm.icon" v-popover:iconListPopover :readonly="true" placeholder="菜单图标名称"
                      class="icon-list__input"></el-input>
          </el-col>
          <el-col :span="2" class="icon-list__tips">
            <el-tooltip placement="top" effect="light">
              <div slot="content">全站推荐使用SVG Sprite, 详细请参考:<a href="" target="_blank">icons/index.js</a>描述</div>
              <i class="el-icon-warning"></i>
            </el-tooltip>
          </el-col>
        </el-row>
      </el-form-item>
      <el-form-item label="状态" size="mini" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">正常</el-radio>
          <el-radio :label="1">禁用</el-radio>
        </el-radio-group>
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
    import Icon from '@/icons'

    export default {
      data () {
        var validateUrl = (rule, value, callback) => {
          if (this.dataForm.type === 1 && !/\S/.test(value)) {
            callback(new Error('菜单URL不能为空'))
          } else {
            callback()
          }
        }
        return {
          visible: false,
          dataForm: {
            id: 0,
            type: 1,
            typeList: ['目录', '菜单', '按钮'],
            name: '',
            description: '',
            parentId: 0,
            parentName: '',
            url: '',
            seq: 0,
            icon: '',
            iconList: [],
            status: 0
          },
          dataRule: {
            name: [
                        {required: true, message: '菜单名称不能为空', trigger: 'blur'}
            ],
            parentName: [
                        {required: true, message: '上级菜单不能为空', trigger: 'change'}
            ],
            url: [
                        {validator: validateUrl, trigger: 'blur'}
            ]
          },
          menuList: [],
          menuListTreeProps: {
            label: 'name',
            children: 'children'
          }
        }
      },
      created () {
        this.iconList = Icon.getNameList()
      },
      methods: {
        init (id) {
          this.dataForm.id = id || 0
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
            this.menuList = []
            this.$http({
              url: this.$http.adornUrl('/sys/menu/select'),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              this.menuList = treeDataTranslate(data.data, 'id', 'parentId')
              console.log('menuList', data.data)
              console.log('menuList', this.menuList)
            }).then(() => {
              console.log('id:', this.dataForm.id)
              if (this.dataForm.id) {
                this.$http({
                  url: this.$http.adornUrl(`/sys/menu/info/${this.dataForm.id}`),
                  method: 'get',
                  params: this.$http.adornParams()
                }).then(({data}) => {
                  console.log('dataForm', data)
                  this.dataForm.id = data.data.id
                  this.dataForm.type = data.data.resourceType
                  this.dataForm.name = data.data.name
                  this.dataForm.description = data.data.description
                  this.dataForm.parentId = data.data.parentId
                  this.dataForm.url = data.data.url
                  this.dataForm.seq = data.data.seq
                  this.dataForm.icon = data.data.icon
                  this.dataForm.status = data.data.status
                  this.menuListTreeSetCurrentNode()
                })
              }
            })
          })
        },
            // 菜单树选中
        menuListTreeCurrentChangeHandle (data, node) {
          this.dataForm.parentId = data.id
          this.dataForm.parentName = data.name
        },
            // 菜单树设置当前选中节点
        menuListTreeSetCurrentNode () {
          this.$refs.menuListTree.setCurrentKey(this.dataForm.parentId)
          this.dataForm.parentName = (this.$refs.menuListTree.getCurrentNode() || {})['name']
        },
            // 图标选中
        iconActiveHandle (iconName) {
          this.dataForm.icon = iconName
        },
            // 表单提交
        dataFormSubmit () {
          this.$refs['dataForm'].validate((valid) => {
            if (valid) {
              this.$http({
                url: this.$http.adornUrl(`/sys/menu/${!this.dataForm.id ? 'save' : 'update'}`),
                method: 'post',
                data: this.$http.adornData({
                  'id': this.dataForm.id || undefined,
                  'resourceType': this.dataForm.type,
                  'name': this.dataForm.name,
                  'description': this.dataForm.description,
                  'parentId': this.dataForm.parentId,
                  'url': this.dataForm.url,
                  'seq': this.dataForm.seq,
                  'icon': this.dataForm.icon,
                  'status': this.dataForm.status
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
        }
      }
    }
</script>

<style lang="scss">
  .mod-menu {
    .menu-list__input,
    .icon-list__input {
      > .el-input__inner {
        cursor: pointer;
      }
    }

    &__icon-popover {
      max-width: 370px;
    }

    &__icon-list {
      max-height: 180px;
      padding: 0;
      margin: -8px 0 0 -8px;

      > .el-button {
        padding: 8px;
        margin: 8px 0 0 8px;

        > span {
          display: inline-block;
          vertical-align: middle;
          width: 18px;
          height: 18px;
          font-size: 18px;
        }
      }
    }

    .icon-list__tips {
      font-size: 18px;
      text-align: center;
      color: #e6a23c;
      cursor: pointer;
    }
  }
  .tree {
    max-height: 250px;
    overflow-y: auto;
  }
  /*滚动条整体部分*/
  .tree::-webkit-scrollbar {
    width: 6px;
    background-color: rgba(217, 217, 217, 0.3);
  }

  /*滚动条里面的上下移动方块*/
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
