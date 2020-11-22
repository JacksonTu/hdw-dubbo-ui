<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
    <el-row>
      <el-col :span="12">
        <el-form-item label="标题" prop="title">
          <el-input v-model="dataForm.title" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="消息类型" prop="msgCategory">
          <el-select v-model="dataForm.msgCategory" placeholder="" value="" style="width:100%">
            <el-option label="通知公告" value="1" />
            <el-option label="系统消息" value="2" />
          </el-select>
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="12">
        <el-form-item label="开始时间" prop="startTime">
          <el-date-picker
            v-model="dataForm.startTime"
            type="datetime"
            placeholder="开始时间"
            value-format="yyyy-MM-dd HH:mm:ss"
          />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="结束时间" prop="endTime">
          <el-date-picker
            v-model="dataForm.endTime"
            type="datetime"
            placeholder="结束时间"
            value-format="yyyy-MM-dd HH:mm:ss"
          />
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="12">
        <el-form-item label="优先级" prop="priority">
          <el-select v-model="dataForm.priority" placeholder="" value="" style="width:100%">
            <el-option label="高" value="H" />
            <el-option label="中" value="M" />
            <el-option label="低" value="L" />
          </el-select>
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item label="通告对象" prop="msgType">
          <el-select v-model="dataForm.msgType" placeholder="" value="" style="width:100%">
            <el-option label="全体用户" value="ALL" />
            <el-option label="指定用户" value="USER" />
          </el-select>
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="12">
        <el-form-item label="摘要" prop="msgAbstract">
          <el-input v-model="dataForm.msgAbstract" type="textarea" :autosize="{ minRows: 2, maxRows: 4}" />
        </el-form-item>
      </el-col>
      <el-col :span="12">
        <el-form-item v-if="dataForm.msgType === 'USER' " label="指定用户" prop="userIds">
          <treeselect v-model="dataForm.userIdArray" :multiple="true" :disable-branch-nodes="true" :show-count="true" placeholder="指定用户" :options="dataForm.users" />
        </el-form-item>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="24">
        <el-form-item label="内容" prop="msgContent">
          <el-input v-model="dataForm.msgContent" type="textarea" :autosize="{ minRows: 5, maxRows: 10}" />
        </el-form-item>
      </el-col>
    </el-row>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import Treeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
export default {
  name: 'NoticeAddOrUpdate',
  components: {Treeselect},
  data () {
    return {
      visible: false,
      dataForm: {
        id: 0,
        title: '',
        msgAbstract: '',
        msgContent: '',
        startTime: '',
        endTime: '',
        priority: '',
        msgCategory: '',
        msgType: '',
        sendStatus: '0',
        delFlag: '0',
        userIds: '',
        userIdArray: [],
        users: []
      },
      dataRule: {
        title: [
          {required: true, message: '标题不能为空', trigger: 'blur'}
        ],
        msgAbstract: [
          {required: true, message: '摘要不能为空', trigger: 'blur'}
        ],
        startTime: [
          {required: true, message: '开始时间不能为空', trigger: 'blur'}
        ],
        endTime: [
          {required: true, message: '结束时间不能为空', trigger: 'blur'}
        ],
        priority: [
          {required: true, message: '优先级不能为空', trigger: 'blur'}
        ],
        msgCategory: [
          {required: true, message: '消息类型不能为空', trigger: 'blur'}
        ],
        msgType: [
          {required: true, message: '通告对象类型不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  mounted () {
  },
  activated () {
  },
  methods: {
    init (id) {
      this.dataForm.id = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        this.initUser()
        if (this.dataForm.id) {
          this.$http({
            url: this.$http.adornUrl(`/notice/info/${this.dataForm.id}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.title = data.data.title
              this.dataForm.msgAbstract = data.data.msgAbstract
              this.dataForm.msgContent = data.data.msgContent
              this.dataForm.startTime = data.data.startTime
              this.dataForm.endTime = data.data.endTime
              this.dataForm.priority = data.data.priority
              this.dataForm.msgCategory = data.data.msgCategory
              this.dataForm.msgType = data.data.msgType
              this.dataForm.sendStatus = data.data.sendStatus
              this.dataForm.delFlag = data.data.delFlag
              this.dataForm.userIds = data.data.userIds
              this.setNotice(data.data)
            }
          })
        }
      })
    },
    initUser () {
      this.dataForm.users = []
      this.$http({
        url: this.$http.adornUrl(`/notice/treeUser`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          console.log('user:', data.data)
          this.dataForm.users = data.data
        }
      })
    },
    setNotice (val) {
      // key遍历
      Object.keys(this.dataForm).map(key => {
        this.dataForm[key] = val[key]
      })
      // 字符串转数组
      if (this.dataForm.userIds) {
        this.dataForm.userIdArray = this.dataForm.userIds.split(',')
      }
    },
    // 表单提交
    dataFormSubmit () {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          if (this.dataForm.msgType === 'USER' && !this.dataForm.userIdArray) {
            this.$message({
              message: '指定用户不能为空',
              type: 'warning'
            })
            return
          }
          // 数组转字符串
          if (this.dataForm.userIdArray) {
            this.dataForm.userIds = this.dataForm.userIdArray.join(',')
          }
          this.$http({
            url: this.$http.adornUrl(`/notice/${!this.dataForm.id ? 'save' : 'update'}`),
            method: 'post',
            data: this.$http.adornData({
              'id': this.dataForm.id || undefined,
              'title': this.dataForm.title,
              'msgAbstract': this.dataForm.msgAbstract,
              'msgContent': this.dataForm.msgContent,
              'startTime': this.dataForm.startTime,
              'endTime': this.dataForm.endTime,
              'priority': this.dataForm.priority,
              'msgCategory': this.dataForm.msgCategory,
              'msgType': this.dataForm.msgType,
              'sendStatus': this.dataForm.sendStatus,
              'delFlag': this.dataForm.delFlag,
              'userIds': this.dataForm.userIds
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
