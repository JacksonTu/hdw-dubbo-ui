<template>
  <nav class="site-navbar" :class="'site-navbar--' + navbarLayoutType">
    <div class="site-navbar__header">
      <h1 class="site-navbar__brand" @click="$router.push({ name: 'home' })">
        <a class="site-navbar__brand-lg" href="javascript:;">基本平台</a>
        <a class="site-navbar__brand-mini" href="javascript:;">BS</a>
      </h1>
    </div>
    <div class="site-navbar__body clearfix">
      <el-menu
        class="site-navbar__menu"
        mode="horizontal">
        <el-menu-item class="site-navbar__switch" index="0" @click="sidebarFold = !sidebarFold">
          <icon-svg name="zhedie"></icon-svg>
        </el-menu-item>
      </el-menu>
      <el-menu
        class="site-navbar__menu site-navbar__menu--right"
        mode="horizontal">
        <el-menu-item index="1" @click="$router.push({ name: 'theme' })">
          <template slot="title">
            <el-badge>
              <icon-svg name="shezhi" class="el-icon-setting"></icon-svg>
            </el-badge>
          </template>
        </el-menu-item>
        <el-menu-item  class="site-navbar__avatar" index="2">
          <i class="el-icon-rank" title="浏览器内全屏"></i>
        </el-menu-item>
        <el-menu-item class="site-navbar__avatar" index="3">
          <message></message>
        </el-menu-item>
        <el-menu-item class="site-navbar__avatar" index="4">
          <el-dropdown :show-timeout="0" placement="bottom">
            <span class="el-dropdown-link">
              <img src="~@/assets/img/avatar.png" :alt="userName">{{ userName }}
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item @click.native="updatePasswordHandle()">修改密码</el-dropdown-item>
              <el-dropdown-item @click.native="logoutHandle()">退出</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </el-menu-item>
      </el-menu>
    </div>
    <!-- 弹窗, 修改密码 -->
    <update-password v-if="updatePassowrdVisible" ref="updatePassowrd"></update-password>
  </nav>
</template>

<script>
    import UpdatePassword from './main-navbar-update-password'
    import {clearLoginInfo} from '@/utils'
    import message from './main-message'

    export default {
      data () {
        return {
          updatePassowrdVisible: false
        }
      },
      components: {
        UpdatePassword,
        message
      },
      computed: {
        navbarLayoutType: {
          get () {
            return this.$store.state.common.navbarLayoutType
          }
        },
        sidebarFold: {
          get () {
            return this.$store.state.common.sidebarFold
          },
          set (val) {
            this.$store.commit('common/updateSidebarFold', val)
          }
        },
        mainTabs: {
          get () {
            return this.$store.state.common.mainTabs
          },
          set (val) {
            this.$store.commit('common/updateMainTabs', val)
          }
        },
        userName: {
          get () {
            return this.$store.state.user.name
          }
        },
        userId: {
          get () {
            return this.$store.state.user.id
          }
        }
      },
      created () {
      },
      destroyed () {
      },
      methods: {
            // 修改密码
        updatePasswordHandle () {
          this.updatePassowrdVisible = true
          this.$nextTick(() => {
            this.$refs.updatePassowrd.init()
          })
        },
            // 退出
        logoutHandle () {
          this.$confirm('确定进行[退出]操作?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            this.$http({
              url: this.$http.adornUrl('/sys/logout'),
              method: 'post',
              data: this.$http.adornData()
            }).then(({data}) => {
              if (data && data.code === 0) {
                clearLoginInfo()
                this.$router.push({name: 'login'})
              }
            })
          }).catch(() => {
          })
        }
      }
    }
</script>

<style scoped>
  .messageHide {
    position: absolute !important;
    top: 14px !important;
    right: -20px !important;
    background: #d00 !important;
    height: 16px !important;
    line-height: 16px !important;
    font-size: 12px !important;
    -webkit-border-radius: 50px !important;
    -moz-border-radius: 50px !important;
    border-radius: 50px !important;
    padding: 0 8px !important;
    color: #fff !important;
  }
</style>
