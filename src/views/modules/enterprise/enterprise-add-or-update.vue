<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="120px">
      <el-form-item label="企业名称" prop="enterpriseName">
        <el-input v-model="dataForm.enterpriseName" placeholder="企业名称"></el-input>
      </el-form-item>
      <el-form-item label="企业注册码" prop="businessLicenseNumber">
        <el-input v-model="dataForm.businessLicenseNumber" placeholder="工商注册码"></el-input>
      </el-form-item>
      <el-form-item label="企业编号" prop="enterpriseCode">
        <el-input v-model="dataForm.enterpriseCode" placeholder="企业编号"></el-input>
      </el-form-item>
      <el-form-item label="所属行业" prop="industryCode">
        <el-select v-model="dataForm.industryCode" placeholder="请选择">
          <el-option
            v-for="item in industryList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
            <span style="float: left">{{ item.name }}</span>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="所属区域" prop="areaCode">
        <el-select v-model="dataForm.areaCode" placeholder="请选择">
          <el-option
            v-for="item in areaList"
            :key="item.id"
            :label="item.name"
            :value="item.id">
            <span style="float: left">{{ item.name }}</span>
          </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="企业类型" prop="enterpriseType">
        <el-radio-group v-model="dataForm.enterpriseType">
          <el-radio :label="0">国企</el-radio>
          <el-radio :label="1">民企</el-radio>
          <el-radio :label="2">私企</el-radio>
          <el-radio :label="3">外企</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="企业联系电话" prop="telephone">
        <el-input v-model="dataForm.telephone" placeholder="企业联系电话"></el-input>
      </el-form-item>
      <el-form-item label="企业邮箱" prop="email">
        <el-input v-model="dataForm.email" placeholder="企业邮箱"></el-input>
      </el-form-item>
      <el-form-item label="邮政编码" prop="zipCode">
        <el-input v-model="dataForm.zipCode" placeholder="邮政编码"></el-input>
      </el-form-item>
      <el-form-item label="法人" prop="legalPerson">
        <el-input v-model="dataForm.legalPerson" placeholder="法人"></el-input>
      </el-form-item>
      <el-form-item label="企业负责人姓名" prop="mainPerson">
        <el-input v-model="dataForm.mainPerson" placeholder="企业负责人姓名"></el-input>
      </el-form-item>
      <el-form-item label="企业负责人电话" prop="mainPersonMobile">
        <el-input v-model="dataForm.mainPersonMobile" placeholder="企业负责人移动电话号码"></el-input>
      </el-form-item>
      <el-form-item label="x坐标" prop="mapX">
        <el-input v-model="dataForm.mapX" placeholder="x坐标"></el-input>
      </el-form-item>
      <el-form-item label="y坐标" prop="mapY">
        <el-input v-model="dataForm.mapY" placeholder="y坐标"></el-input>
      </el-form-item>
      <el-form-item label="z坐标" prop="mapZ">
        <el-input v-model="dataForm.mapZ" placeholder="z坐标"></el-input>
      </el-form-item>
      <el-form-item label="地址" prop="address">
        <el-input v-model="dataForm.address" placeholder="地址"></el-input>
      </el-form-item>
      <el-form-item label="企业状态" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">正常</el-radio>
          <el-radio :label="1">禁用</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="企业附件" prop="">
        <el-upload
          class="upload-demo"
          ref="upload"
          :action="uploadUrl"
          name="file"
          accept=".doc,.docx,.xls,.xlsx,.pdf,.png,.jpg,.gif"
          :on-preview="handlePreview"
          :on-remove="handleRemove"
          :before-upload="beforeUpload"
          :on-success="handleSuccess"
          :file-list="fileList"
          :auto-upload="false">
          <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
          <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button>
          <div slot="tip" class="el-upload__tip">只能上传doc/docx/xls/xlsx/pdf/png/jpg/gif文件，且不超过100MB</div>
        </el-upload>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
    export default {
      data () {
        return {
          visible: false,
          dataForm: {
            id: 0,
            businessLicenseNumber: '',
            enterpriseCode: '',
            enterpriseName: '',
            industryCode: '',
            areaCode: '',
            enterpriseType: 0,
            telephone: '',
            email: '',
            zipCode: '',
            legalPerson: '',
            mainPerson: '',
            mainPersonMobile: '',
            mapX: '',
            mapY: '',
            mapZ: '',
            address: '',
            status: 0
          },
          industryList: [],
          areaList: [],
          uploadUrl: '',
          fileList: [],
          dataRule: {
            businessLicenseNumber: [
              {required: true, message: '企业注册码(工商注册码-三证合一)不能为空', trigger: 'blur'}
            ],
            enterpriseCode: [
              {required: true, message: '企业编号不能为空', trigger: 'blur'}
            ],
            enterpriseName: [
              {required: true, message: '企业名称不能为空', trigger: 'blur'}
            ],
            industryCode: [
              {required: true, message: '所属行业不能为空', trigger: 'blur'}
            ],
            areaCode: [
              {required: true, message: '所属区域不能为空', trigger: 'blur'}
            ],
            enterpriseType: [
              {required: true, message: '企业类型不能为空', trigger: 'blur'}
            ],
            telephone: [
              {required: true, message: '企业联系电话不能为空', trigger: 'blur'},
              {pattern: /^1[34578]\d{9}$/, message: '你的手机号格式不正确'}
            ],
            email: [
              {required: true, message: '企业邮箱不能为空', trigger: 'blur'},
              { pattern: /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/, message: '你的邮箱格式不正确', trigger: 'blur' }
            ],
            zipCode: [
              {required: true, message: '邮政编码不能为空', trigger: 'blur'}
            ],
            legalPerson: [
              {required: true, message: '法人不能为空', trigger: 'blur'}
            ],
            mainPerson: [
              {required: true, message: '企业负责人姓名不能为空', trigger: 'blur'}
            ],
            mainPersonMobile: [
              {required: true, message: '企业负责人移动电话号码不能为空', trigger: 'blur'},
              {pattern: /^1[34578]\d{9}$/, message: '你的手机号格式不正确'}
            ],
            mapX: [
              {required: true, message: 'x坐标不能为空', trigger: 'blur'}
            ],
            mapY: [
              {required: true, message: 'y坐标不能为空', trigger: 'blur'}
            ],
            mapZ: [
              {required: true, message: 'z坐标不能为空', trigger: 'blur'}
            ],
            address: [
              {required: true, message: '地址不能为空', trigger: 'blur'}
            ],
            status: [
              {required: true, message: '企业状态不能为空', trigger: 'blur'}
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
            this.uploadUrl = this.$http.adornUrl(`/enterprise/uploadFile?token=` + this.$cookie.get('token'))
            this.fileList = []
            this.lookFile(id)
            this.initIndustryList()
            this.initAreaList()
            if (this.dataForm.id) {
              this.$http({
                url: this.$http.adornUrl(`/enterprise/info/${this.dataForm.id}`),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.dataForm.businessLicenseNumber = data.data.businessLicenseNumber
                  this.dataForm.enterpriseCode = data.data.enterpriseCode
                  this.dataForm.enterpriseName = data.data.enterpriseName
                  this.dataForm.industryCode = data.data.industryCode
                  this.dataForm.areaCode = data.data.areaCode
                  this.dataForm.enterpriseType = data.data.enterpriseType
                  this.dataForm.telephone = data.data.telephone
                  this.dataForm.email = data.data.email
                  this.dataForm.zipCode = data.data.zipCode
                  this.dataForm.legalPerson = data.data.legalPerson
                  this.dataForm.mainPerson = data.data.mainPerson
                  this.dataForm.mainPersonMobile = data.data.mainPersonMobile
                  this.dataForm.mapX = data.data.mapX
                  this.dataForm.mapY = data.data.mapY
                  this.dataForm.mapZ = data.data.mapZ
                  this.dataForm.address = data.data.address
                  this.dataForm.status = data.data.status
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
                url: this.$http.adornUrl(`/enterprise/${!this.dataForm.id ? 'save' : 'update'}`),
                method: 'post',
                data: this.$http.adornData({
                  'id': this.dataForm.id || undefined,
                  'businessLicenseNumber': this.dataForm.businessLicenseNumber,
                  'enterpriseCode': this.dataForm.enterpriseCode,
                  'enterpriseName': this.dataForm.enterpriseName,
                  'industryCode': this.dataForm.industryCode,
                  'areaCode': this.dataForm.areaCode,
                  'enterpriseType': this.dataForm.enterpriseType,
                  'telephone': this.dataForm.telephone,
                  'email': this.dataForm.email,
                  'zipCode': this.dataForm.zipCode,
                  'legalPerson': this.dataForm.legalPerson,
                  'mainPerson': this.dataForm.mainPerson,
                  'mainPersonMobile': this.dataForm.mainPersonMobile,
                  'mapX': this.dataForm.mapX,
                  'mapY': this.dataForm.mapY,
                  'mapZ': this.dataForm.mapZ,
                  'address': this.dataForm.address,
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
        },
        submitUpload () {
          this.$refs.upload.submit()
        },
        handleRemove (file, fileList) {
          console.log(file, fileList)
          this.deleteFile(file)
        },
        handlePreview (file) {
          console.log(file)
          window.open(file.url, '_blank').location
        },
        beforeUpload (file) {
          const isLt100M = file.size / 1024 / 1024 < 100
          if (!isLt100M) {
            this.$message.error('上传文件大小不能超过 100MB!')
          }
          return isLt100M
        },
        handleSuccess (response, file, fileList) {
          console.log(response)
          if (response && response.code === 0) {
            file.name = response.data.name
            file.url = response.data.url
          }
          console.log(file)
          console.log(fileList)
        },
        // 删除文件
        deleteFile (file) {
          this.$http({
            url: this.$http.adornUrl('/enterprise/deleteFile'),
            method: 'get',
            params: this.$http.adornParams({
              'id': this.dataForm.id,
              'name': file.name,
              'url': file.url
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: data.msg,
                type: 'success'
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        },
        // 查看文件
        lookFile (id) {
          this.$http({
            url: this.$http.adornUrl('/enterprise/lookFile/' + id),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.fileList = data.data
            }
          })
        }
      }
    }
</script>
