<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input v-model="dataForm.tableName" placeholder="表名" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button type="primary" @click="generatorCodeHandle()"
                   :disabled="dataListSelections.length <= 0">批量生成
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
        prop="tableName"
        header-align="center"
        align="center"
        label="表名">
      </el-table-column>
      <el-table-column
        prop="engine"
        header-align="center"
        align="center"
        label="Engine">
      </el-table-column>
      <el-table-column
        prop="tableComment"
        header-align="center"
        align="center"
        label="表备注">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
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
</template>

<script>
    export default {
      data () {
        return {
          dataForm: {
            tableName: ''
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
      },
      activated () {
        this.getDataList()
      },
      methods: {
            // 获取数据列表
        getDataList () {
          this.dataListLoading = true
          this.$http({
            url: this.$http.adornUrl('/generator/list'),
            method: 'get',
            params: this.$http.adornParams({
              'page': this.pageIndex,
              'limit': this.pageSize,
              'tableName': this.dataForm.tableName
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
        // 生成代码
        generatorCodeHandle (tableName) {
          let tableNames = tableName ? [tableName] : this.dataListSelections.map(item => {
            return item.tableName
          })
          this.$confirm(`确定对[tableName=${tableNames.join(',')}]进行[${tableName ? '生成代码' : '批量生成代码'}]操作?`, '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            window.location.href = this.$http.adornUrl(`/generator/code?tableNames=` + tableNames.toString() + '&token=' + this.$cookie.get('token'))
          })
        }
      }
    }
</script>
