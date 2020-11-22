<template>
  <nav class="site-navbar" :class="'site-navbar--' + navbarLayoutType">
    <div class="site-navbar__header">
      <h1 class="site-navbar__brand" @click="$router.push({ name: 'home' })">
        <a class="site-navbar__brand-lg" href="javascript:;">hdw-dubbo</a>
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
        <el-menu-item class="site-navbar__avatar" index="1">
          <message title="消息"></message>
        </el-menu-item>
        <el-menu-item class="site-navbar__avatar" index="2"  @click="screen">
          <i class="el-icon-full-screen" :title="fullTitle"></i>
        </el-menu-item>
        <el-menu-item index="3" @click="$router.push({ name: 'theme' })">
          <template slot="title">
            <el-badge>
              <icon-svg name="shezhi" class="el-icon-setting"></icon-svg>
            </el-badge>
          </template>
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
    let flag = 0
    export default {
      data () {
        return {
          updatePassowrdVisible: false,
          fullTitle: '全屏'
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
        // 切换浏览器窗口大小
        screen: function () {
          if (flag === 0) {
            this.fullTitle = '退出全屏'
            this.fullScreen()
            flag = 1
          } else {
            this.fullTitle = '全屏'
            this.exitFullScreen()
            flag = 0
          }
        },
        // 点击放大浏览器窗口
        fullScreen: function () {
          const el = document.documentElement
          const rfs = el.requestFullScreen || el.webkitRequestFullScreen || el.mozRequestFullScreen || el.msRequestFullScreen
          let wscript

          if (typeof rfs !== 'undefined' && rfs) {
            rfs.call(el)
            return
          }

          if (typeof window.ActiveXObject !== 'undefined') {
            wscript = window.ActiveXObject('WScript.Shell')
            if (wscript) {
              wscript.SendKeys('{F11}')
            }
          }
        },
        // 点击还原浏览器窗口
        exitFullScreen: function () {
          const el = document
          const cfs = el.cancelFullScreen || el.webkitCancelFullScreen || el.mozCancelFullScreen || el.exitFullScreen
          let wscript

          if (typeof cfs !== 'undefined' && cfs) {
            cfs.call(el)
            return
          }

          if (typeof window.ActiveXObject !== 'undefined') {
            wscript = window.ActiveXObject('WScript.Shell')
            if (wscript != null) {
              wscript.SendKeys('{F11}')
            }
          }
        },
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
