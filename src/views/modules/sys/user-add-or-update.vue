<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-tabs v-model="activeName" @tab-click="handleClick">
      <el-tab-pane label="用户信息" name="first">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
                 label-width="80px">
          <el-form-item label="用户类型" prop="userType">
            <el-radio-group v-model="dataForm.userType">
              <el-radio v-for="(userType, index) in dataForm.typeList" :label="index" :key="index">{{ userType }}
              </el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="登录名" prop="loginName">
            <el-input v-model="dataForm.loginName" placeholder="登录帐号"></el-input>
          </el-form-item>
          <el-form-item label="用户名" prop="name">
            <el-input v-model="dataForm.name" placeholder="登录帐号"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password" :class="{ 'is-required': !dataForm.id }">
            <el-input v-model="dataForm.password" type="password" placeholder="密码"></el-input>
          </el-form-item>
          <el-form-item label="确认密码" prop="comfirmPassword" :class="{ 'is-required': !dataForm.id }">
            <el-input v-model="dataForm.comfirmPassword" type="password" placeholder="确认密码"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="dataForm.email" placeholder="邮箱"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="phone">
            <el-input v-model="dataForm.phone" placeholder="手机号"></el-input>
          </el-form-item>
          <el-form-item label="性别" size="mini" prop="sex">
            <el-radio-group v-model="dataForm.sex">
              <el-radio :label="0">男</el-radio>
              <el-radio :label="1">女</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="状态" size="mini" prop="status">
            <el-radio-group v-model="dataForm.status">
              <el-radio :label="0">正常</el-radio>
              <el-radio :label="1">禁用</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="是否过期" size="mini" prop="expired">
            <el-radio-group v-model="dataForm.expired">
              <el-radio :label="0">是</el-radio>
              <el-radio :label="1">否</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
      </el-tab-pane>
      <el-tab-pane label="分配角色" name="second" :disabled="!dataForm.id">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
                 label-width="80px">
          <el-form-item label="角色" size="mini" prop="roleIdList">
            <el-checkbox-group v-model="dataForm.roleIdList">
              <el-checkbox v-for="role in roleList" :key="role.id" :label="role.id">{{ role.name }}</el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-form>
      </el-tab-pane>
      <el-tab-pane label="企业信息" name="third" :disabled="!dataForm.id || dataForm.userType!=1">
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
          <el-form-item label="所属部门" prop="departmentId">
            <SelectTree
              :props="props"
              :options="deptOptions"
              :value="valueId"
              :clearable="isClearable"
              :accordion="isAccordion"
              @getValue="getValue($event)"
            />
          </el-form-item>
          <el-form-item label="所属职务" prop="jobName">
            <el-select clearable filterable v-model="dataForm.jobId" placeholder="请选择">
              <el-option
                v-for="item in jobList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="是否领导" size="mini" prop="isLeader">
            <el-radio-group v-model="dataForm.isLeader">
              <el-radio :label="0">是</el-radio>
              <el-radio :label="1">否</el-radio>
            </el-radio-group>
          </el-form-item>
        </el-form>
      </el-tab-pane>
      <el-tab-pane label="分配企业" name="fourth" :disabled="!dataForm.id || dataForm.userType!=2">
        <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
                 label-width="80px">
          <el-form-item label="分配企业" prop="enterpriseIdList">
            <el-select clearable filterable multiple v-model="dataForm.enterpriseIdList" placeholder="请选择"
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
        </el-form>
      </el-tab-pane>
    </el-tabs>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import {isEmail, isMobile} from '@/utils/validate'
  import {treeDataTranslate} from '@/utils'
  import SelectTree from '../../../components/tree-select/index'

  export default {
    data() {
      var validatePassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('密码不能为空'))
        } else {
          callback()
        }
      }
      var validateComfirmPassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('确认密码不能为空'))
        } else if (this.dataForm.password !== value) {
          callback(new Error('确认密码与密码输入不一致'))
        } else {
          callback()
        }
      }
      var validateEmail = (rule, value, callback) => {
        if (!isEmail(value)) {
          callback(new Error('邮箱格式错误'))
        } else {
          callback()
        }
      }
      var validateMobile = (rule, value, callback) => {
        if (!isMobile(value)) {
          callback(new Error('手机号格式错误'))
        } else {
          callback()
        }
      }
      return {
        visible: false,
        dataForm: {
          id: 0,
          loginName: '',
          name: '',
          password: '',
          comfirmPassword: '',
          sex: 0,
          email: '',
          phone: '',
          roleIdList: [],
          status: 1,
          expired: 1,
          isLeader: 1,
          userType: 1,
          typeList: ['超级用户', '企业用户', '监管用户'],
          enterpriseId: 0,
          departmentId: 0,
          jobId: 0,
          enterpriseIdList: ''
        },
        activeName: 'first',
        roleList: [],
        unitList: [],
        jobList: [],
        deptOptions: [],
        isClearable: true,      // 可清空（可选）
        isAccordion: true,      // 可收起（可选）
        valueId: '',            // 初始ID（可选）
        props: {                // 配置项（必选）
          value: 'id',
          label: 'name',
          children: 'children'
        },
        dataRule: {
          loginName: [
            {required: true, message: '登录名不能为空', trigger: 'blur'}
          ],
          name: [
            {required: true, message: '用户名不能为空', trigger: 'blur'}
          ],
          password: [
            {validator: validatePassword, trigger: 'blur'}
          ],
          comfirmPassword: [
            {validator: validateComfirmPassword, trigger: 'blur'}
          ],
          email: [
            {required: true, message: '邮箱不能为空', trigger: 'blur'},
            {validator: validateEmail, trigger: 'blur'}
          ],
          phone: [
            {required: true, message: '手机号不能为空', trigger: 'blur'},
            {validator: validateMobile, trigger: 'blur'}
          ]
        }
      }
    },
    components: {
      SelectTree
    },
    methods: {
      init(id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.activeName = 'first'
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          this.roleList = []
          this.unitList = []
          this.deptOptions = []
          this.jobList = []
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/user/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.loginName = data.data.loginName
                this.dataForm.name = data.data.name
                this.dataForm.sex = data.data.sex
                this.dataForm.email = data.data.email
                this.dataForm.phone = data.data.phone
                this.dataForm.roleIdList = data.data.roleIdList
                this.dataForm.status = data.data.status
                this.dataForm.expired = data.data.expired
                this.dataForm.userType = data.data.userType
                this.dataForm.isLeader = data.data.isLeader
                this.dataForm.enterpriseId = data.data.enterpriseId
                this.dataForm.departmentId = data.data.departmentId
                this.dataForm.jobId = data.data.jobId
                this.dataForm.enterpriseIdList = data.data.enterpriseIdList
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit() {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/user/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'loginName': this.dataForm.loginName,
                'name': this.dataForm.name,
                'password': this.dataForm.password,
                'email': this.dataForm.email,
                'phone': this.dataForm.phone,
                'status': this.dataForm.status,
                'roleIdList': this.dataForm.roleIdList,
                'expired': this.dataForm.expired,
                'isLeader': this.dataForm.isLeader,
                'userType': this.dataForm.userType,
                'enterpriseId': this.dataForm.enterpriseId,
                'departmentId': this.dataForm.departmentId,
                'jobId': this.dataForm.jobId,
                'enterpriseIdList': this.dataForm.userType === 2 ? this.dataForm.enterpriseIdList : []
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
      // tab切换
      handleClick(tab, event) {
        if (tab.name === 'first') {

        } else if (tab.name === 'second') {
          this.getRoleList()
        } else if (tab.name === 'third') {
          this.getUnitList()
          this.getDeptList()
          this.getJobList()
        } else if (tab.name === 'fourth') {
          this.getUnitList()
        }
      },
      // 获取角色
      getRoleList() {
        this.$http({
          url: this.$http.adornUrl('/sys/role/select'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.roleList = data && data.code === 0 ? data.data : []
        })
      },
      // 获取企业树
      getUnitList() {
        this.$http({
          url: this.$http.adornUrl('/enterprise/getEnterpriseTree'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.unitList = data.data
        })
      },
      // 获取企业部门树
      getDeptList() {
        this.dataForm.enterpriseId = this.dataForm.enterpriseId || ''
        this.deptList = []
        this.$http({
          url: this.$http.adornUrl('/enterprise/enterpriseDepartment/getDeptSelectTree?enterpriseId=' + this.dataForm.enterpriseId),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.deptOptions = treeDataTranslate(data.data)
        })
      },
      // 获取部门职位树
      getJobList() {
        this.dataForm.departmentId = this.dataForm.departmentId || ''
        this.jobList = []
        this.$http({
          url: this.$http.adornUrl('/enterprise/enterpriseJob/selectJobTree?deptId=' + this.dataForm.departmentId),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.jobList = data.data
        })
      },
      // 企业选择改变
      handleChangeEnterprise() {
        this.getDeptList()
      },
      // 部门选择改变
      handleChangeDept() {
        this.getJobList()
      },
      // 取值
      getValue(value) {
        this.valueId = value
        this.dataForm.departmentId = this.valueId
        this.handleChangeDept()
      }
    }
  }
</script>
