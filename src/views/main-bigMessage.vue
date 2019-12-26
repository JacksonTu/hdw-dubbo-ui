<template>
  <el-dialog title="消息" class="msgDialog" width="70%" :visible.sync="visible">
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="未读" name="first">
        <el-date-picker
          v-model="timeRange"
          type="datetimerange"
          value-format="yyyy-MM-dd HH:mm:ss"
          :picker-options="pickerOptions"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          align="right">
        </el-date-picker>
        <el-button @click="getDataList(1)">查询</el-button>
        <el-button type="danger" @click="updateStatusHandle()" v-if="isAuth('sms/smsRecord/delete')"
                   :disabled="dataListSelections.length <= 0">批量已读
        </el-button>
        <el-table
          :data="dataList"
          border
          v-loading="dataListLoading"
          @selection-change="selectionChangeHandle"
          style="width: 100%;margin-top: 15px;">
          <el-table-column
            type="selection"
            header-align="center"
            align="center"
            width="50">
          </el-table-column>
          <el-table-column show-overflow-tooltip
                           prop="title"
                           header-align="center"
                           align="center"
                           label="消息标题">
          </el-table-column>
          <el-table-column show-overflow-tooltip
                           prop="content"
                           header-align="center"
                           align="center"
                           label="消息内容">
          </el-table-column>
          <el-table-column
            prop="pushTime"
            header-align="center"
            align="center"
            label="推送时间">
          </el-table-column>
          <el-table-column
            prop="status"
            header-align="center"
            align="center"
            label="推送状态	">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.status === 0" size="small">待推送</el-tag>
              <el-tag v-if="scope.row.status === 1" size="small">成功</el-tag>
              <el-tag v-if="scope.row.status === 2" size="small" type="danger">失败</el-tag>
              <el-tag v-if="scope.row.status === 3" size="small">已读</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            fixed="right"
            header-align="center"
            align="center"
            width="150"
            label="操作">
            <template slot-scope="scope">
              <el-button type="text" size="small" v-if="isAuth('sms/smsRecord/delete')"
                         @click="updateStatusHandle(scope.row.id)"><i class="el-icon-edit"></i></el-button>
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
      </el-tab-pane>
      <el-tab-pane label="所有" name="second">
        <el-date-picker
          v-model="timeRange"
          type="datetimerange"
          value-format="yyyy-MM-dd HH:mm:ss"
          :picker-options="pickerOptions"
          range-separator="至"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          align="right">
        </el-date-picker>
        <el-button @click="getDataList(1)">查询</el-button>
        <el-button type="danger" @click="deleteHandle" v-if="isAuth('sms/smsRecord/delete')"
                   :disabled="dataListSelections.length <= 0">批量删除
        </el-button>
        <el-table
          :data="dataList"
          border
          v-loading="dataListLoading"
          @selection-change="selectionChangeHandle"
          style="width: 100%;margin-top: 15px;">
          <el-table-column
            type="selection"
            header-align="center"
            align="center"
            width="50">
          </el-table-column>
          <el-table-column show-overflow-tooltip
                           prop="title"
                           header-align="center"
                           align="center"
                           label="消息标题">
          </el-table-column>
          <el-table-column show-overflow-tooltip
                           prop="content"
                           header-align="center"
                           align="center"
                           label="消息内容">
          </el-table-column>
          <el-table-column
            prop="pushTime"
            header-align="center"
            align="center"
            label="推送时间">
          </el-table-column>
          <el-table-column
            prop="status"
            header-align="center"
            align="center"
            label="推送状态	">
            <template slot-scope="scope">
              <el-tag v-if="scope.row.status === 0" size="small">待推送</el-tag>
              <el-tag v-if="scope.row.status === 1" size="small">成功</el-tag>
              <el-tag v-if="scope.row.status === 2" size="small" type="danger">失败</el-tag>
              <el-tag v-if="scope.row.status === 3" size="small">已读</el-tag>
            </template>
          </el-table-column>
          <el-table-column
            fixed="right"
            header-align="center"
            align="center"
            width="150"
            label="操作">
            <template slot-scope="scope">
              <el-button type="text" size="small" v-if="isAuth('sms/smsRecord/delete')"
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
      </el-tab-pane>
    </el-tabs>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        activeName: 'first',
        timeRange: [],
        dataForm: {
          startTime: '',
          endTime: '',
          status: -1
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        pickerOptions: {
          shortcuts: [{
            text: '最近一周',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7)
              picker.$emit('pick', [start, end])
            }
          }, {
            text: '最近一个月',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30)
              picker.$emit('pick', [start, end])
            }
          }, {
            text: '最近三个月',
            onClick (picker) {
              const end = new Date()
              const start = new Date()
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90)
              picker.$emit('pick', [start, end])
            }
          }]
        }
      }
    },
    computed: {
      userId: {
        get () { return this.$store.state.user.id }
      }
    },
    components: {},
    methods: {
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/sms/smsRecord/delete'),
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
      // 获取起止时间
      getTimeTange () {
        if (this.timeRange && this.timeRange.length === 2) {
          this.dataForm.startTime = this.timeRange[0]
          this.dataForm.endTime = this.timeRange[1]
        }
      },
      // 获取数据列表
      getDataList (pageIndex) {
        if (pageIndex) {
          this.pageIndex = pageIndex
        }
        this.getTimeTange()
        this.visible = true
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/sms/smsRecord/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'startTime': this.dataForm.startTime,
            'endTime': this.dataForm.endTime,
            'status': this.dataForm.status,
            'userId': this.userId
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            console.log(data.data)
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
      // 已读
      updateStatusHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定进行[${id ? '已读' : '批量已读'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$emit('bigRead', ids)
        })
      },
      // 未读消息列表
      getUnread () {
        this.dataForm.status = -1
        this.getDataList()
      },
      // 所有消息列表
      getAll () {
        this.dataForm.status = ''
        this.getDataList()
      },
      handleClick () {
        this.timeRange = []
        this.dataForm.startTime = ''
        this.dataForm.endTime = ''
        if (this.activeName === 'first') {
          this.getUnread()
        } else {
          this.getAll()
        }
      }
    }
  }
</script>
<style>
  .v-modal {
    z-index: 1000 !important;
  }
</style>
