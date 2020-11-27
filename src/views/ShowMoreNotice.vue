<template>
  <el-dialog
    :title="title"
    :width="width"
    top="50px"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    :visible.sync="isVisible"
    append-to-body
  >
    <div class="mod-config">
      <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList(1)">
        <el-form-item>
          <el-input v-model="dataForm.title" placeholder="标题" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button @click="getDataList(1)">查询</el-button>
          <el-button type="success" @click="readAll()">一键已读</el-button>
        </el-form-item>
      </el-form>
      <el-table
        :data="dataList"
        border
        v-loading="dataListLoading"
        @selection-change="selectionChangeHandle"
        style="width: 100%;">
        <el-table-column
          type="selection"
          header-align="center"
          align="center"
          width="50">
        </el-table-column>
        <el-table-column
          prop="title"
          header-align="center"
          align="center"
          label="标题">
        </el-table-column>
        <el-table-column
          prop="msgAbstract"
          header-align="center"
          align="center"
          label="摘要"
          show-overflow-tooltip>
        </el-table-column>
        <el-table-column
          prop="msgContent"
          header-align="center"
          align="center"
          label="内容"
          show-overflow-tooltip>
        </el-table-column>
        <el-table-column
          prop="msgCategory"
          header-align="center"
          align="center"
          :filters="[{ text: '通知公告', value: '1' }, { text: '系统消息', value: '2' }]"
          :filter-method="filterTag"
          label="消息类型">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.msgCategory ==='1'" size="small" type="success">通知公告</el-tag>
            <el-tag v-else-if="scope.row.msgCategory ==='2'" size="small" type="primary">系统消息</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="createUser"
          header-align="center"
          align="center"
          label="发布人">
        </el-table-column>
        <el-table-column
          prop="priority"
          header-align="center"
          align="center"
          label="优先级">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.priority ==='H'" size="small" type="success">高</el-tag>
            <el-tag v-else-if="scope.row.priority ==='M'" size="small" type="primary">中</el-tag>
            <el-tag v-else-if="scope.row.priority ==='L'" size="small" type="info">低</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="readFlag"
          header-align="center"
          align="center"
          label="阅读状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.readFlag ==='0'" size="small" type="success">未读</el-tag>
            <el-tag v-else-if="scope.row.readFlag ==='1'" size="small" type="primary">已读</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          prop="sendTime"
          header-align="center"
          align="center"
          label="发布时间">
        </el-table-column>
        <el-table-column
          fixed="right"
          header-align="center"
          align="center"
          width="80"
          label="操作">
          <template slot-scope="scope">
            <el-button v-if="scope.row.readFlag ==='0'" type="text" size="small"
                       @click="updateNoticeStatus(scope.row)"><i class="el-icon-setting table-operation"></i>
            </el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="sizeChangeHandle"
        @current-change="currentChangeHandle"
        :current-page="pageIndex"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="pageSize"
        :total="totalPage"
        layout="total, sizes, prev, pager, next, jumper">
      </el-pagination>
    </div>
    <span slot="footer" class="dialog-footer">
      <el-button plain :loading="buttonLoading" @click="isVisible = false">取消</el-button>
    </span>
  </el-dialog>

</template>

<script>
export default {
  name: 'ShowMoreNotice',
  components: {},
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: ''
    },
    type: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      screenWidth: 0,
      buttonLoading: false,
      width: this.initWidth(),
      dataForm: {
        title: ''
      },
      dataList: [],
      pageIndex: 1,
      pageSize: 10,
      totalPage: 0,
      dataListLoading: false,
      dataListSelections: [],
      addOrUpdateVisible: false
    }
  },
  computed: {
    isVisible: {
      get () {
        return this.dialogVisible
      },
      set () {
        this.close()
      }
    }
  },
  mounted () {
    this.getDataList()
    window.onresize = () => {
      return (() => {
        this.width = this.initWidth()
      })()
    }
  },
  activated () {
  },
  methods: {
    initWidth () {
      this.screenWidth = document.body.clientWidth
      if (this.screenWidth < 991) {
        return '90%'
      } else if (this.screenWidth < 1400) {
        return '70%'
      } else {
        return '60%'
      }
    },
    updateNoticeStatus (record) {
      this.$http({
        url: this.$http.adornUrl(`/notice/send/editByNoticeIdAndUserId/${record.noticeId}`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '已读成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList()
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    readAll () {
      this.$http({
        url: this.$http.adornUrl('/notice/send/readAll'),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '一键已读成功',
            type: 'success',
            duration: 1500,
            onClose: () => {
              this.getDataList()
            }
          })
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    close () {
      this.$emit('close')
    },
    // 获取数据列表
    getDataList (pageIndex) {
      if (pageIndex) {
        this.pageIndex = pageIndex
      }
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/notice/send/getMyNoticeSend'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'title': this.dataForm.title
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
          // console.log(data.data)
          this.dataList = data.data.list
          this.totalPage = data.data.totalCount
        } else {
          this.dataList = []
          this.totalPage = 0
        }
        this.dataListLoading = false
      })
    },
    // 每页数
    sizeChangeHandle (val) {
      this.pageSize = val
      this.pageIndex = 1
      this.getDataList()
    },
    // 当前页
    currentChangeHandle (val) {
      this.pageIndex = val
      this.getDataList()
    },
    // 多选
    selectionChangeHandle (val) {
      this.dataListSelections = val
    },
    filterTag (value, row) {
      return row.msgCategory === value
    }
  }
}
</script>

<style scoped>

</style>
