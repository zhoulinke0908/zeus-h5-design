<template>
  <div id="app">
    <!-- 导航首页 -->
    <div v-if="page === 'home'" class="home-page">
      <div class="home-header">
        <h1>Zeus H5 设计稿</h1>
        <p>Vant 2 版本 · 移动端适配</p>
      </div>
      <div class="home-cards">
        <div class="home-card" @click="page = 'login'">
          <van-icon name="friends-o" size="28" color="#F19871" />
          <div class="card-title">登录 / 注册</div>
          <div class="card-desc">密码登录 · 验证码登录 · 注册</div>
        </div>
        <div class="home-card" @click="page = 'settings'">
          <van-icon name="setting-o" size="28" color="#7C6FF7" />
          <div class="card-title">设置</div>
          <div class="card-desc">修改姓名 · 手机号 · 密码</div>
        </div>
        <div class="home-card" @click="page = 'my'">
          <van-icon name="user-o" size="28" color="#F5A623" />
          <div class="card-title">我的</div>
          <div class="card-desc">个人主页 · 菜单 · 导航</div>
        </div>
      </div>
    </div>

    <!-- 登录 -->
    <LoginPage v-if="page === 'login'" @back="page = 'home'" />

    <!-- 设置 -->
    <MySettingsPage
      v-if="page === 'settings'"
      :user.sync="user"
      @back="page = 'home'"
    />

    <!-- 我的 -->
    <MyPage
      v-if="page === 'my'"
      :user="user"
      :unread="3"
      @goSettings="page = 'settings'"
      @goMy="page = 'my'"
    />
  </div>
</template>

<script>
import LoginPage from './Login.vue'
import MySettingsPage from './MySettings.vue'
import MyPage from './My.vue'

export default {
  name: 'App',
  components: { LoginPage, MySettingsPage, MyPage },
  data() {
    return {
      page: 'home',
      user: {
        name: '周林科',
        phone: '13650020342',
        company: '融查查科技',
        logo: 'https://zeus-h5-test.irongchacha.cn/static/img/rcckjlogo.896b174e.png',
        department: '业务部',
        roles: ['业务经理', '按揭专员', '财务人员', '按揭总监', '方案审批员', '管理员']
      }
    }
  }
}
</script>

<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif;
  -webkit-font-smoothing: antialiased;
}

.home-page {
  min-height: 100vh;
  background: linear-gradient(180deg, #FFF8F5, #F5F5F7);
  padding: 60px 20px;
  text-align: center;
}
.home-header h1 { font-size: 28px; color: #1D1D1F; }
.home-header p { font-size: 14px; color: #86868B; margin: 8px 0 36px; }
.home-cards {
  display: flex; flex-wrap: wrap; gap: 16px;
  justify-content: center; max-width: 520px; margin: 0 auto;
}
.home-card {
  width: 140px; padding: 24px 16px; background: #fff;
  border-radius: 16px; cursor: pointer;
  box-shadow: 0 2px 12px rgba(0,0,0,0.06);
  transition: transform 0.2s;
}
.home-card:active { transform: scale(0.96); }
.card-title { font-size: 16px; font-weight: 600; color: #1D1D1F; margin: 10px 0 4px; }
.card-desc { font-size: 12px; color: #86868B; }
</style>
