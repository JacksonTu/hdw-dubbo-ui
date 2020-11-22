<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList(1)">
      <el-form-item>
        <el-input v-model="dataForm.title" placeholder="标题" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList(1)">查询</el-button>
        <el-button v-if="isAuth('notice/save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
        <el-button v-if="isAuth('notice/delete')" type="danger" @click="deleteHandle()"
                   :disabled="dataListSelections.length <= 0">批量删除
        </el-button>
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
        prop="msgType"
        header-align="center"
        align="center"
        label="通告对象">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.msgType ==='ALL'" size="small" type="success">全体用户</el-tag>
          <el-tag v-else-if="scope.row.msgType ==='USER'" size="small" type="primary">指定用户</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="sendStatus"
        header-align="center"
        align="center"
        label="发布状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.sendStatus ==='0'" size="small" type="primary">未发布</el-tag>
          <el-tag v-else-if="scope.row.sendStatus ==='1'" size="small" type="success">已发布</el-tag>
          <el-tag v-else-if="scope.row.sendStatus ==='2'" size="small" type="info">已撤销</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="sendTime"
        header-align="center"
        align="center"
        label="发布时间">
      </el-table-column>
      <el-table-column
        prop="cancelTime"
        header-align="center"
        align="center"
        label="撤销时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="scope.row.sendStatus ==='0' && isAuth('notice/update')" type="text" size="small"
                     @click="addOrUpdateHandle(scope.row.id)"><i class="el-icon-edit"></i></el-button>
          <el-button v-if="scope.row.sendStatus ==='0' && isAuth('notice/release')" type="text" size="small"
                     @click="releaseData(scope.row.id)"><i class="el-icon-s-promotion"></i></el-button>
          <el-button v-if="scope.row.sendStatus ==='1' && isAuth('notice/revoke')" type="text" size="small"
                     @click="revokeData(scope.row.id)"><i class="el-icon-s-release"></i></el-button>
          <el-button v-if="scope.row.sendStatus !=='1' && isAuth('notice/delete')" type="text" size="small"
                     @click="deleteHandle(scope.row.id)"><i class="el-icon-delete"></i></el-button>
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
    <!-- 弹窗, 新增 / 修改 -->
    <notice-add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></notice-add-or-update>
  </div>
</template>

<script>
import NoticeAddOrUpdate from './sysNotice-add-or-update'

export default {
  name: 'NoticeManage',
  data () {
    return {
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
  components: {
    NoticeAddOrUpdate
  },
  activated () {
    this.getDataList()
  },
  methods: {
    // 获取数据列表
    getDataList (pageIndex) {
      if (pageIndex) {
        this.pageIndex = pageIndex
      }
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/notice/list'),
        method: 'get',
        params: this.$http.adornParams({
          'page': this.pageIndex,
          'limit': this.pageSize,
          'title': this.dataForm.title,
        })
      }).then(({data}) => {
        if (data && data.code === 0) {
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
    // 新增 / 修改
    addOrUpdateHandle (id) {
      this.addOrUpdateVisible = true
      this.$nextTick(() => {
        this.$refs.addOrUpdate.init(id)
      })
    },
    // 删除
    deleteHandle (id) {
      var ids = id ? [id] : this.dataListSelections.map(item => {
        return item.id
      })
      this.$confirm(`确定要进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/notice/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.$message({
              message: '操作成功',
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
      })
    },
    releaseData (id) {
      this.$http({
        url: this.$http.adornUrl(`/notice/release/${id}`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '发布成功',
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
    revokeData (id) {
      this.$http({
        url: this.$http.adornUrl(`/notice/revoke/${id}`),
        method: 'get',
        params: this.$http.adornParams()
      }).then(({data}) => {
        if (data && data.code === 0) {
          this.$message({
            message: '撤销成功',
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
    }
  }
}
</script>
