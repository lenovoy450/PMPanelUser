<template>
  <div class="main">
    <a-form id="formLogin" class="user-layout-login" ref="formLogin" :form="form" @submit="handleSubmit">
      <div style="margin-top: 30px">
        <a-alert v-if="isLoginError" type="error" showIcon style="margin-bottom: 24px;" :message="$t('login.failed')" />
        <a-form-item>
          <a-input size="large" type="text" :placeholder="$t('login.email')" v-decorator="['username', {rules: [{ required: true, type: 'email', message: $t('register.emailMessage') }], validateTrigger: ['change', 'blur']}]">
            <a-icon slot="prefix" type="mail" :style="{ color: 'rgba(0,0,0,.25)' }"/>
          </a-input>
        </a-form-item>

        <a-form-item>
          <a-input-password size="large" :placeholder="$t('register.password')" v-decorator="['password',{rules: [{ required: true, message: $t('login.passwordMessage') }], validateTrigger: 'blur'}]">
            <a-icon slot="prefix" type="lock" :style="{ color: 'rgba(0,0,0,.25)' }"/>
          </a-input-password>
        </a-form-item>
      </div>

      <a-form-item>
        <a-checkbox v-decorator="['rememberMe', { valuePropName: 'checked', initialValue: true }]">{{ $t('login.rememberMe') }}</a-checkbox>
        <router-link :to="{ path:'/auth/forgot' }" class="forge-password" style="float: right;"> {{ $t('login.forgetPass') }} </router-link>
        <router-link :to="{ path:'/auth/register' }" class="register" style="margin-right: 10px;float: right;"> {{ $t('login.register') }} </router-link>
      </a-form-item>

      <a-form-item style="margin-top:24px">
        <a-button
          size="large"
          type="primary"
          htmlType="submit"
          class="login-button"
          :loading="state.loginBtn"
          :disabled="state.loginBtn"> {{ $t('login.loginBtn') }} </a-button>
      </a-form-item>

      <select-lang style="margin-top: -20px;float: right" />
    </a-form>
  </div>
</template>

<script>
// import md5 from 'md5'
// import TwoStepCaptcha from '@/components/tools/TwoStepCaptcha'
import { mapActions } from 'vuex'
import storage from 'store'
import { ACCESS_TOKEN } from '@/store/mutation-types'

export default {
  components: {
    // TwoStepCaptcha
  },
  data () {
    return {
      customActiveKey: 'tab1',
      loginBtn: false,
      // auth type: 0 email, 1 username, 2 telephone
      loginType: 0,
      isLoginError: false,
      requiredTwoStepCaptcha: false,
      stepCaptchaVisible: false,
      form: this.$form.createForm(this),
      state: {
        time: 60,
        loginBtn: false,
        // auth type: 0 email, 1 username, 2 telephone
        loginType: 0,
        smsSendBtn: false
      }
    }
  },
  created () {
    const token = this.getCookie('token')
    if (token.length > 40) {
      this.$i18n.locale === 'zh-CN' ? this.$message.success('正在跳转') : this.$message.success('Redirect~')
      storage.set(ACCESS_TOKEN, token, (this.getCookie('expire_in')) * 1000)
      document.cookie = 'token=;-1'
      this.$router.push({ path: '/' })
    }
    const token2 = this.getCookie('ip')
    if (token2.length > 40) {
      this.$i18n.locale === 'zh-CN' ? this.$message.success('正在跳转') : this.$message.success('Redirect~')
      storage.set(ACCESS_TOKEN, token2, (this.getCookie('expire_in')) * 1000)
      document.cookie = 'token=;-1'
      this.$router.push({ path: '/' })
    }
  },
  methods: {
    ...mapActions(['Login']),
    // handler
    handleUsernameOrEmail (rule, value, callback) {
      const { state } = this
      const regex = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+((\.[a-zA-Z0-9_-]{2,3}){1,2})$/
      if (regex.test(value)) {
        state.loginType = 0
      } else {
        state.loginType = 1
      }
      callback()
    },
    handleSubmit (e) {
      e.preventDefault()
      const {
        form: { validateFields },
        state,
        customActiveKey,
        Login
      } = this

      state.loginBtn = true

      const validateFieldsKey = customActiveKey === 'tab1' ? ['username', 'password', 'rememberMe'] : ['mobile', 'captcha']

      validateFields(validateFieldsKey, { force: true }, (err, values) => {
        if (!err) {
          console.log('auth form', values)
          const loginParams = { ...values }
          delete loginParams.username
          loginParams[!state.loginType ? 'email' : 'username'] = values.username
          // loginParams.password = md5(values.password)
          loginParams.password = values.password
          if (values.rememberMe === undefined) {
            loginParams.rememberMe = false
          } else {
            loginParams.rememberMe = values.rememberMe
          }
          Login(loginParams)
            .then((res) => this.loginSuccess(res))
            .catch(err => this.requestFailed(err))
            .finally(() => {
              state.loginBtn = false
            })
        } else {
          setTimeout(() => {
            state.loginBtn = false
          }, 600)
        }
      })
    },
    loginSuccess (res) {
      // check res.homePage define, set $router.push name res.homePage
      // Why not enter onComplete
      /*
      this.$router.push({ name: 'analysis' }, () => {
        console.log('onComplete')
        this.$notification.success({
          message: '欢迎',
          description: `${timeFix()}，欢迎回来`
        })
      })
      */
      this.$router.push({ path: '/' })
      // 延迟 1 秒显示欢迎信息
      setTimeout(() => {
        this.$notification.success({
          message: this.$t('login.successWelcome'),
          description: this.$t('login.successInfo')
        })
      }, 1000)
      this.isLoginError = false
    },
    requestFailed () {
      this.isLoginError = true
    },
    // 获取cookie
    getCookie: function (cname) {
      const name = cname + '='
      const ca = document.cookie.split(';')
      for (let i = 0; i < ca.length; i++) {
        let c = ca[i]
        while (c.charAt(0) === ' ') c = c.substring(1)
        if (c.indexOf(name) !== -1) {
          return c.substring(name.length, c.length)
        }
      }
      return ''
    }
  }
}
</script>

<style lang="less" scoped>
.user-layout-login {
  label {
    font-size: 14px;
  }

  .getCaptcha {
    display: block;
    width: 100%;
    height: 40px;
  }

  .forge-password {
    font-size: 14px;
  }

  button.login-button {
    padding: 0 15px;
    font-size: 16px;
    height: 40px;
    width: 100%;
  }
}
</style>
