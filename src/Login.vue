<template>
  <div class="login-page">
    <!-- Logo -->
    <div class="logo-area">
      <van-image :src="logoUrl" width="48" height="48" fit="contain" />
      <div class="logo-text">融查查科技</div>
    </div>

    <!-- Tabs -->
    <van-tabs v-model="activeTab" :color="PRIMARY" :line-width="28" :line-height="3" title-active-color="#1D1D1F" title-inactive-color="#86868B">
      <van-tab title="登录"></van-tab>
      <van-tab title="注册"></van-tab>
    </van-tabs>

    <!-- Login Panel -->
    <div v-show="activeTab === 0" class="form-card">
      <div class="mode-switch" @click="loginMode = loginMode === 'password' ? 'sms' : 'password'">
        {{ loginMode === 'password' ? '验证码登陆' : '密码登录' }}
      </div>

      <!-- Password Login -->
      <template v-if="loginMode === 'password'">
        <van-field
          v-model="loginForm.username"
          label="用户名"
          placeholder="请输入用户名"
          left-icon="user-o"
          :error="loginErrors.username"
          :error-message="loginErrors.username ? '请输入用户名' : ''"
          @input="loginErrors.username = false"
        />
        <van-field
          v-model="loginForm.password"
          :type="showLoginPwd ? 'text' : 'password'"
          label="密码"
          placeholder="请输入密码"
          left-icon="lock"
          :right-icon="showLoginPwd ? 'eye-o' : 'closed-eye'"
          :error="loginErrors.password"
          :error-message="loginErrors.password ? '请输入密码' : ''"
          @click-right-icon="showLoginPwd = !showLoginPwd"
          @input="loginErrors.password = false"
        />
      </template>

      <!-- SMS Login -->
      <template v-else>
        <van-field
          v-model="smsLoginForm.phone"
          label="手机号"
          placeholder="请输入手机号"
          left-icon="phone-o"
          type="tel"
          maxlength="11"
          :error="smsLoginErrors.phone"
          :error-message="smsLoginErrors.phone ? '请输入正确的11位手机号' : ''"
          @input="smsLoginForm.phone = smsLoginForm.phone.replace(/\D/g, '')"
        />
        <div class="sms-row">
          <van-field
            v-model="smsLoginForm.code"
            label="验证码"
            placeholder="短信验证码"
            left-icon="comment-o"
            maxlength="6"
            :error="smsLoginErrors.code"
            :error-message="smsLoginErrors.code ? '请输入验证码' : ''"
            @input="smsLoginForm.code = smsLoginForm.code.replace(/\D/g, '')"
          />
          <van-button
            :type="smsLoginCount > 0 ? 'default' : 'primary'"
            size="small"
            :disabled="smsLoginCount > 0"
            class="sms-btn"
            @click="sendSmsLoginCode"
          >
            {{ smsLoginCount > 0 ? smsLoginCount + 's' : '获取验证码' }}
          </van-button>
        </div>
      </template>

      <!-- Slider -->
      <SliderVerify ref="loginSlider" />

      <van-button round block :color="PRIMARY" class="submit-btn" @click="handleLogin">
        {{ loginMode === 'password' ? '登录' : '验证码登录' }}
      </van-button>
    </div>

    <!-- Register Panel -->
    <div v-show="activeTab === 1" class="form-card">
      <van-field
        v-model="regForm.name"
        label="用户姓名"
        placeholder="请输入用户姓名"
        left-icon="user-o"
        maxlength="20"
        :error="regErrors.name"
        :error-message="regErrors.name ? '请输入用户姓名' : ''"
        @input="regErrors.name = false"
      />
      <van-field
        v-model="regForm.phone"
        label="手机号"
        placeholder="请输入手机号"
        left-icon="phone-o"
        type="tel"
        maxlength="11"
        :error="regErrors.phone"
        :error-message="regErrors.phone ? '请输入正确的11位手机号' : ''"
        @input="regForm.phone = regForm.phone.replace(/\D/g, '')"
      />
      <div class="sms-row">
        <van-field
          v-model="regForm.code"
          label="验证码"
          placeholder="短信验证码"
          left-icon="comment-o"
          maxlength="6"
          :error="regErrors.code"
          :error-message="regErrors.code ? '请输入验证码' : ''"
          @input="regForm.code = regForm.code.replace(/\D/g, '')"
        />
        <van-button
          :type="regSmsCount > 0 ? 'default' : 'primary'"
          size="small"
          :disabled="regSmsCount > 0"
          class="sms-btn"
          @click="sendRegSmsCode"
        >
          {{ regSmsCount > 0 ? regSmsCount + 's' : '获取验证码' }}
        </van-button>
      </div>
      <van-field
        v-model="regForm.password"
        :type="showRegPwd ? 'text' : 'password'"
        label="设置密码"
        placeholder="请输入密码（6-20位）"
        left-icon="lock"
        :right-icon="showRegPwd ? 'eye-o' : 'closed-eye'"
        maxlength="20"
        :error="regErrors.password"
        :error-message="regErrors.password ? '密码长度需为6-20位' : ''"
        @click-right-icon="showRegPwd = !showRegPwd"
        @input="regErrors.password = false"
      />
      <van-field
        v-model="regForm.confirm"
        :type="showRegCpwd ? 'text' : 'password'"
        label="确认密码"
        placeholder="请再次输入密码"
        left-icon="lock"
        :right-icon="showRegCpwd ? 'eye-o' : 'closed-eye'"
        maxlength="20"
        :error="regErrors.confirm"
        :error-message="regErrors.confirm ? '两次输入的密码不一致' : ''"
        @click-right-icon="showRegCpwd = !showRegCpwd"
        @input="regErrors.confirm = false"
      />

      <SliderVerify ref="regSlider" />

      <div class="agreement" @click="regForm.agreed = !regForm.agreed">
        <van-icon :name="regForm.agreed ? 'checked' : 'checked'" :color="regForm.agreed ? PRIMARY : '#C7C7CC'" size="18" />
        <span>同意<span class="link" @click.stop="$emit('showAgreement')">《用户服务协议》</span></span>
      </div>

      <van-button round block :color="PRIMARY" class="submit-btn" @click="handleRegister">
        注册
      </van-button>
    </div>
  </div>
</template>

<script>
import SliderVerify from './SliderVerify.vue'

const PRIMARY = '#F19871'

export default {
  name: 'LoginPage',
  components: { SliderVerify },
  data() {
    return {
      PRIMARY,
      logoUrl: 'https://zeus-h5-test.irongchacha.cn/static/img/rcckjlogo.896b174e.png',
      activeTab: 0,
      loginMode: 'password',

      // Password login
      loginForm: { username: '', password: '' },
      loginErrors: { username: false, password: false },
      showLoginPwd: false,

      // SMS login
      smsLoginForm: { phone: '', code: '' },
      smsLoginErrors: { phone: false, code: false },
      smsLoginCount: 0,
      smsLoginTimer: null,

      // Register
      regForm: { name: '', phone: '', code: '', password: '', confirm: '', agreed: false },
      regErrors: { name: false, phone: false, code: false, password: false, confirm: false },
      showRegPwd: false,
      showRegCpwd: false,
      regSmsCount: 0,
      regSmsTimer: null
    }
  },
  beforeDestroy() {
    clearInterval(this.smsLoginTimer)
    clearInterval(this.regSmsTimer)
  },
  methods: {
    // SMS login code
    sendSmsLoginCode() {
      if (!/^1[3-9]\d{9}$/.test(this.smsLoginForm.phone)) {
        this.smsLoginErrors.phone = true
        this.$toast('请输入正确的手机号')
        return
      }
      this.smsLoginCount = 60
      this.smsLoginTimer = setInterval(() => {
        this.smsLoginCount--
        if (this.smsLoginCount <= 0) clearInterval(this.smsLoginTimer)
      }, 1000)
      this.$toast('验证码已发送')
    },

    // Login
    handleLogin() {
      if (this.loginMode === 'password') {
        this.loginErrors.username = !this.loginForm.username
        this.loginErrors.password = !this.loginForm.password
        if (!this.loginForm.username || !this.loginForm.password) return
      } else {
        this.smsLoginErrors.phone = !/^1[3-9]\d{9}$/.test(this.smsLoginForm.phone)
        this.smsLoginErrors.code = !/^\d{4,6}$/.test(this.smsLoginForm.code)
        if (this.smsLoginErrors.phone || this.smsLoginErrors.code) return
      }
      if (!this.$refs.loginSlider.isDone()) {
        this.$toast('请完成滑块验证')
        return
      }
      this.$toast('登录成功')
    },

    // Register SMS
    sendRegSmsCode() {
      if (!/^1[3-9]\d{9}$/.test(this.regForm.phone)) {
        this.regErrors.phone = true
        this.$toast('请输入正确的手机号')
        return
      }
      this.regSmsCount = 60
      this.regSmsTimer = setInterval(() => {
        this.regSmsCount--
        if (this.regSmsCount <= 0) clearInterval(this.regSmsTimer)
      }, 1000)
      this.$toast('验证码已发送')
    },

    // Register
    handleRegister() {
      const { name, phone, code, password, confirm, agreed } = this.regForm
      this.regErrors.name = !name || name.length < 2
      this.regErrors.phone = !/^1[3-9]\d{9}$/.test(phone)
      this.regErrors.code = !/^\d{4,6}$/.test(code)
      this.regErrors.password = password.length < 6 || password.length > 20
      this.regErrors.confirm = password !== confirm
      if (Object.values(this.regErrors).some(v => v)) return
      if (!this.$refs.regSlider.isDone()) { this.$toast('请完成滑块验证'); return }
      if (!agreed) { this.$toast('请先同意用户协议'); return }
      this.$toast('注册成功')
    }
  }
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  background: linear-gradient(180deg, #FFF8F5, #F5F5F7);
  padding-bottom: 40px;
}
.logo-area {
  display: flex; flex-direction: column; align-items: center;
  padding: 40px 0 16px;
}
.logo-text {
  margin-top: 8px; font-size: 13px; color: #86868B;
}

/* Fix Vant tab font */
::v-deep .van-tab { font-size: 18px; font-weight: 600; }
::v-deep .van-tabs__line { border-radius: 3px; }

.form-card {
  margin: 16px 20px 0;
  background: #fff; border-radius: 16px;
  padding: 20px 16px 24px;
  box-shadow: 0 2px 12px rgba(0,0,0,0.04);
}

.mode-switch {
  text-align: right; font-size: 13px; color: #F19871;
  padding-bottom: 8px; cursor: pointer;
}

::v-deep .van-field__control { font-size: 15px; }
::v-deep .van-field__label { font-size: 14px; color: #1D1D1F; font-weight: 500; }

.sms-row {
  display: flex; align-items: center;
}
.sms-row ::v-deep .van-field { flex: 1; }
.sms-btn {
  flex-shrink: 0; margin-right: 16px;
  height: 36px; font-size: 13px;
}

.submit-btn {
  margin-top: 28px;
  height: 48px; font-size: 17px; letter-spacing: 1px;
}

.agreement {
  display: flex; align-items: center; gap: 6px;
  margin-top: 18px; font-size: 13px; color: #86868B; cursor: pointer;
}
.link { color: #F19871; }
</style>
