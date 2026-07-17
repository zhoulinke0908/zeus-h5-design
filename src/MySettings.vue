<template>
  <div class="settings-page">
    <!-- Nav -->
    <van-nav-bar title="设置" left-arrow @click-left="$emit('back')" :border="false" />

    <!-- Avatar Area -->
    <div class="avatar-area">
      <div class="avatar-circle">{{ user.name.charAt(0) }}</div>
      <div class="avatar-info">
        <div class="display-name">{{ user.name }}</div>
        <div class="display-dept" v-if="user.department">{{ user.department }}</div>
      </div>
    </div>

    <!-- Account Info -->
    <div class="section-label">账号信息</div>
    <van-cell-group>
      <van-cell title="姓名" :value="user.name" icon="user-o" is-link @click="openEdit('nickname')" />
      <van-cell title="手机号" :value="maskPhone(user.phone)" icon="phone-o" is-link @click="openEdit('phone')" />
      <van-cell title="登录密码" value="******" icon="lock" is-link @click="openEdit('password')" />
    </van-cell-group>

    <!-- Logout -->
    <div class="logout-area">
      <van-button round block plain type="danger" @click="showLogout = true">退出登录</van-button>
    </div>

    <!-- Edit Nickname Popup -->
    <van-popup v-model="showNickname" position="bottom" round :style="{ padding: '24px 20px' }">
      <div class="popup-title">修改姓名</div>
      <van-field v-model="nicknameForm.value" placeholder="请输入姓名" maxlength="20" :error-message="nicknameError" />
      <div class="popup-actions">
        <van-button round block @click="showNickname = false">取消</van-button>
        <van-button round block :color="PRIMARY" @click="saveNickname">确定</van-button>
      </div>
    </van-popup>

    <!-- Edit Phone Popup -->
    <van-popup v-model="showPhone" position="bottom" round :style="{ padding: '24px 20px' }">
      <div class="popup-title">修改手机号</div>
      <div class="popup-sub">当前手机号：{{ maskPhone(user.phone) }}</div>
      <div class="sms-row">
        <van-field v-model="phoneForm.oldSms" placeholder="原手机验证码" maxlength="6" @input="phoneForm.oldSms = phoneForm.oldSms.replace(/\D/g, '')" />
        <van-button size="small" :type="phoneOldCount > 0 ? 'default' : 'primary'" :disabled="phoneOldCount > 0" @click="sendOldSms">
          {{ phoneOldCount > 0 ? phoneOldCount + 's' : '获取验证码' }}
        </van-button>
      </div>
      <div class="popup-divider"></div>
      <van-field v-model="phoneForm.newPhone" placeholder="请输入新手机号" maxlength="11" type="tel" @input="phoneForm.newPhone = phoneForm.newPhone.replace(/\D/g, '')" />
      <div class="sms-row">
        <van-field v-model="phoneForm.newSms" placeholder="新手机验证码" maxlength="6" @input="phoneForm.newSms = phoneForm.newSms.replace(/\D/g, '')" />
        <van-button size="small" :type="phoneNewCount > 0 ? 'default' : 'primary'" :disabled="phoneNewCount > 0" @click="sendNewSms">
          {{ phoneNewCount > 0 ? phoneNewCount + 's' : '获取验证码' }}
        </van-button>
      </div>
      <div class="popup-actions">
        <van-button round block @click="showPhone = false">取消</van-button>
        <van-button round block :color="PRIMARY" @click="savePhone">确定</van-button>
      </div>
    </van-popup>

    <!-- Edit Password Popup -->
    <van-popup v-model="showPassword" position="bottom" round :style="{ padding: '24px 20px' }">
      <div class="popup-title">修改密码</div>
      <van-field v-model="passwordForm.old" type="password" placeholder="请输入原密码" maxlength="20" />
      <van-field v-model="passwordForm.newPwd" type="password" placeholder="请输入新密码（6-20位）" maxlength="20" />
      <van-field v-model="passwordForm.confirm" type="password" placeholder="请再次输入新密码" maxlength="20" />
      <div class="popup-actions">
        <van-button round block @click="showPassword = false">取消</van-button>
        <van-button round block :color="PRIMARY" @click="savePassword">确定</van-button>
      </div>
    </van-popup>

    <!-- Logout Confirm -->
    <van-dialog v-model="showLogout" title="退出登录" message="确定要退出当前账号吗？" show-cancel-button @confirm="$emit('logout')" />
  </div>
</template>

<script>
const PRIMARY = '#F19871'

export default {
  name: 'SettingsPage',
  props: {
    user: {
      type: Object,
      default: () => ({ name: '周林科', phone: '13650020342', department: '' })
    }
  },
  data() {
    return {
      PRIMARY,
      showNickname: false,
      showPhone: false,
      showPassword: false,
      showLogout: false,

      nicknameForm: { value: '' },
      nicknameError: '',

      phoneForm: { oldSms: '', newPhone: '', newSms: '' },
      phoneOldCount: 0,
      phoneNewCount: 0,
      phoneOldTimer: null,
      phoneNewTimer: null,

      passwordForm: { old: '', newPwd: '', confirm: '' }
    }
  },
  beforeDestroy() {
    clearInterval(this.phoneOldTimer)
    clearInterval(this.phoneNewTimer)
  },
  methods: {
    maskPhone(p) {
      if (!p || p.length < 7) return p
      return p.slice(0, 3) + ' **** ' + p.slice(-4)
    },
    openEdit(type) {
      this.nicknameError = ''
      if (type === 'nickname') {
        this.nicknameForm.value = this.user.name
        this.showNickname = true
      } else if (type === 'phone') {
        this.phoneForm = { oldSms: '', newPhone: '', newSms: '' }
        this.showPhone = true
      } else {
        this.passwordForm = { old: '', newPwd: '', confirm: '' }
        this.showPassword = true
      }
    },
    saveNickname() {
      const v = this.nicknameForm.value.trim()
      if (!v || v.length < 2) {
        this.nicknameError = '姓名不能为空'
        return
      }
      this.$emit('update:user', { ...this.user, name: v })
      this.showNickname = false
      this.$toast('姓名修改成功')
    },
    savePhone() {
      const { oldSms, newPhone, newSms } = this.phoneForm
      if (!/^\d{4,6}$/.test(oldSms) || !/^1[3-9]\d{9}$/.test(newPhone) || !/^\d{4,6}$/.test(newSms)) {
        this.$toast('请填写完整信息')
        return
      }
      clearInterval(this.phoneOldTimer)
      clearInterval(this.phoneNewTimer)
      this.$emit('update:user', { ...this.user, phone: newPhone })
      this.showPhone = false
      this.$toast('手机号修改成功')
    },
    savePassword() {
      const { old, newPwd, confirm } = this.passwordForm
      if (!old) return this.$toast('请输入原密码')
      if (newPwd.length < 6 || newPwd.length > 20) return this.$toast('密码长度需为6-20位')
      if (newPwd !== confirm) return this.$toast('两次输入的密码不一致')
      this.showPassword = false
      this.$toast('密码修改成功')
    },
    sendOldSms() {
      this.phoneOldCount = 60
      this.phoneOldTimer = setInterval(() => { this.phoneOldCount--; if (this.phoneOldCount <= 0) clearInterval(this.phoneOldTimer) }, 1000)
      this.$toast('验证码已发送')
    },
    sendNewSms() {
      if (!/^1[3-9]\d{9}$/.test(this.phoneForm.newPhone)) return this.$toast('请输入正确的手机号')
      this.phoneNewCount = 60
      this.phoneNewTimer = setInterval(() => { this.phoneNewCount--; if (this.phoneNewCount <= 0) clearInterval(this.phoneNewTimer) }, 1000)
      this.$toast('验证码已发送')
    }
  }
}
</script>

<style scoped>
.settings-page {
  min-height: 100vh;
  background: linear-gradient(180deg, #FFF8F5 0%, #F5F5F7 40%);
}
::v-deep .van-nav-bar { background: transparent; }
::v-deep .van-nav-bar__title { font-size: 17px; font-weight: 600; }

.avatar-area {
  display: flex; align-items: center; gap: 14px;
  padding: 16px 24px 24px;
}
.avatar-circle {
  width: 56px; height: 56px; border-radius: 50%;
  background: #F19871; display: flex; align-items: center;
  justify-content: center; font-size: 24px; color: #fff; font-weight: 700;
}
.avatar-info { flex: 1; }
.display-name { font-size: 19px; font-weight: 700; color: #1D1D1F; }
.display-dept { font-size: 13px; color: #86868B; margin-top: 2px; }

.section-label {
  padding: 0 20px 8px; font-size: 13px; color: #86868B; font-weight: 500;
}
::v-deep .van-cell-group { margin: 0 12px; border-radius: 16px; overflow: hidden; }
::v-deep .van-cell { font-size: 15px; }
::v-deep .van-cell__value { color: #1D1D1F; }

.logout-area { padding: 32px 20px; }

.popup-title { font-size: 18px; font-weight: 700; text-align: center; margin-bottom: 16px; }
.popup-sub { text-align: center; font-size: 13px; color: #86868B; margin-bottom: 12px; }
.popup-divider { height: 1px; background: #F2F2F5; margin: 10px 0 14px; }
.popup-actions {
  display: flex; gap: 12px; margin-top: 20px;
}
.popup-actions .van-button { flex: 1; }
.sms-row {
  display: flex; align-items: center;
}
.sms-row .van-field { flex: 1; }
.sms-row .van-button { flex-shrink: 0; margin-right: 8px; }
</style>
