<template>
  <div class="mod-config">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-select v-model="dataForm.industryCode" filterable placeholder="行业" clearable>
          <el-option
            v-for="item in industryList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-select v-model="dataForm.areaCode" filterable placeholder="区域" clearable>
          <el-option
            v-for="item in areaList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-input v-model="dataForm.enterpriseName" placeholder="企业名称" clearable></el-input>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('enterprise/enterprise/save')" type="primary" @click="addOrUpdateHandle()">新增
        </el-button>
        <el-button v-if="isAuth('enterprise/enterprise/delete')" type="danger" @click="deleteHandle()"
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
        prop="enterpriseName"
        header-align="center"
        align="center"
        label="企业名称">
      </el-table-column>
      <el-table-column
        prop="businessLicenseNumber"
        header-align="center"
        align="center"
        label="工商注册码">
      </el-table-column>
      <el-table-column
        prop="enterpriseCode"
        header-align="center"
        align="center"
        label="企业编号">
      </el-table-column>
      <el-table-column
        prop="industryName"
        header-align="center"
        align="center"
        label="所属行业">
      </el-table-column>
      <el-table-column
        prop="areaName"
        header-align="center"
        align="center"
        label="所属区域">
      </el-table-column>
      <el-table-column
        prop="enterpriseType"
        header-align="center"
        align="center"
        label="企业类型">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.enterpriseType === 0" size="small">国企</el-tag>
          <el-tag v-if="scope.row.enterpriseType === 1" size="small" type="success">民企</el-tag>
          <el-tag v-if="scope.row.enterpriseType === 2" size="small" type="info">私企</el-tag>
          <el-tag v-if="scope.row.enterpriseType === 3" size="small">外企</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="telephone"
        header-align="center"
        align="center"
        label="企业联系电话">
      </el-table-column>
      <el-table-column
        prop="email"
        header-align="center"
        align="center"
        label="企业邮箱">
      </el-table-column>
      <el-table-column
        prop="zipCode"
        header-align="center"
        align="center"
        label="邮政编码">
      </el-table-column>
      <el-table-column
        prop="legalPerson"
        header-align="center"
        align="center"
        label="法人">
      </el-table-column>
      <el-table-column
        prop="mainPerson"
        header-align="center"
        align="center"
        label="企业负责人姓名">
      </el-table-column>
      <el-table-column
        prop="mainPersonMobile"
        header-align="center"
        align="center"
        label="企业负责人电话">
      </el-table-column>
      <el-table-column
        prop="mapX"
        header-align="center"
        align="center"
        label="x坐标">
      </el-table-column>
      <el-table-column
        prop="mapY"
        header-align="center"
        align="center"
        label="y坐标">
      </el-table-column>
      <el-table-column
        prop="mapZ"
        header-align="center"
        align="center"
        label="z坐标">
      </el-table-column>
      <el-table-column
        prop="address"
        header-align="center"
        align="center"
        label="地址">
      </el-table-column>
      <el-table-column
        prop="status"
        header-align="center"
        align="center"
        label="企业状态">
        <template slot-scope="scope">
          <el-tag v-if="scope.row.status === 0" size="small">正常</el-tag>
          <el-tag v-if="scope.row.status === 1" size="small" type="danger">禁用</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="创建时间">
      </el-table-column>
      <el-table-column
        prop="updateTime"
        header-align="center"
        align="center"
        label="更新时间">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('enterprise/enterprise/update')" type="text" size="small"
                     @click="addOrUpdateHandle(scope.row.id)"><i class="el-icon-edit"></i></el-button>
          <el-button v-if="isAuth('enterprise/enterprise/delete')" type="text" size="small"
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
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
    import AddOrUpdate from './enterprise-add-or-update'

    export default {
      data () {
        return {
          dataForm: {
            industryCode: '',
            areaCode: '',
            enterpriseName: ''
          },
          dataList: [],
          pageIndex: 1,
          pageSize: 10,
          totalPage: 0,
          dataListLoading: false,
          dataListSelections: [],
          addOrUpdateVisible: false,
          industryList: [],
          areaList: []
        }
      },
      components: {
        AddOrUpdate
      },
      activated () {
        this.getDataList()
        this.initIndustryList()
        this.initAreaList()
      },
      methods: {
            // 获取数据列表
        getDataList () {
          this.dataListLoading = true
          this.$http({
            url: this.$http.adornUrl('/enterprise/list'),
            method: 'get',
            params: this.$http.adornParams({
              'page': this.pageIndex,
              'limit': this.pageSize,
              'industryCode': this.dataForm.industryCode,
              'areaCode': this.dataForm.areaCode,
              'enterpriseName': this.dataForm.enterpriseName
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              console.log(data.data.list)
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
          this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$http({
              url: this.$http.adornUrl('/enterprise/delete'),
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
        initIndustryList () {
          this.industryList = []
          this.$http({
            url: this.$http.adornUrl('/sys/dic/select/9'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.industryList = data.data
          })
        },
        initAreaList () {
          this.areaList = []
          this.$http({
            url: this.$http.adornUrl('/sys/dic/select/16'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.areaList = data.data
          })
        }
      }
    }
</script>
