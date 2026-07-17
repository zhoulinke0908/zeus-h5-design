<template>
  <div class="my-page">
    <!-- Header -->
    <div class="profile-header">
      <div class="company-name" v-if="user.company">{{ user.company }}</div>
      <div class="header-row">
        <div class="avatar-box" @click="$emit('goSettings')">
          <img v-if="user.logo" :src="user.logo" alt="logo" class="avatar-img" />
          <span v-else class="avatar-text">{{ user.name.charAt(0) }}</span>
        </div>
        <div class="header-info">
          <div class="user-name">{{ user.name }}</div>
          <div class="user-phone">{{ maskPhone(user.phone) }}</div>
          <div class="user-dept" v-if="user.department">{{ user.department }}</div>
        </div>
        <div class="notify-btn" @click="$emit('goMessages')">
          <van-icon name="bell" size="20" color="#fff" />
          <van-badge :content="unread" v-if="unread" />
        </div>
      </div>
      <div class="tags-row" v-if="roles.length || user.department">
        <span class="tag" v-for="(r, i) in roles" :key="i">{{ r }}</span>
      </div>
      <div class="stats-row">
        <div class="stat"><div class="stat-num">¥0</div><div class="stat-label">累计佣金</div></div>
        <div class="stat"><div class="stat-num">0</div><div class="stat-label">邀约客户</div></div>
        <div class="stat"><div class="stat-num">0</div><div class="stat-label">团队成员</div></div>
      </div>
    </div>

    <!-- Menu -->
    <div class="menu-section">
      <van-cell-group>
        <van-cell title="我的佣金" icon="gold-coin-o" is-link @click="onMenu('我的佣金')" />
        <van-cell title="我的邀约" icon="friends-o" is-link @click="onMenu('我的邀约')" />
        <van-cell title="我的团队" icon="cluster-o" is-link @click="onMenu('我的团队')" />
        <van-cell title="邀请好友" icon="add-o" is-link @click="onMenu('邀请好友')" />
      </van-cell-group>
      <van-cell-group class="menu-gap">
        <van-cell title="关于我们" icon="info-o" :value="'v1.2.0'" is-link @click="onMenu('关于我们')" />
        <van-cell title="设置" icon="setting-o" is-link @click="$emit('goSettings')" />
        <van-cell title="联系我们" icon="phone-o" is-link @click="onMenu('联系我们')" />
        <van-cell title="系统消息" icon="chat-o" is-link @click="onMenu('系统消息')">
          <template #right-icon>
            <van-badge :content="unread" v-if="unread" />
            <van-icon name="arrow" class="arrow-icon" />
          </template>
        </van-cell>
      </van-cell-group>
    </div>

    <!-- Tab Bar -->
    <van-tabbar v-model="activeTab" :active-color="PRIMARY">
      <van-tabbar-item icon="home-o" @click="$emit('goHome')">首页</van-tabbar-item>
      <van-tabbar-item icon="friends-o" @click="$emit('goCustomers')">客户管理</van-tabbar-item>
      <van-tabbar-item icon="search" @click="$emit('goMarket')">拓客空间</van-tabbar-item>
      <van-tabbar-item icon="apps-o" @click="$emit('goProducts')">产品管理</van-tabbar-item>
      <van-tabbar-item icon="user-o" @click="$emit('goMy')">我的</van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<script>
const PRIMARY = '#F19871'

export default {
  name: 'MyPage',
  props: {
    user: {
      type: Object,
      default: () => ({
        name: '周林科',
        phone: '13650020342',
        company: '融查查科技',
        logo: 'https://zeus-h5-test.irongchacha.cn/static/img/rcckjlogo.896b174e.png',
        department: '业务部',
        roles: ['业务经理', '按揭专员', '财务人员', '按揭总监', '方案审批员', '管理员']
      })
    },
    unread: { type: [Number, String], default: 0 }
  },
  data() {
    return { PRIMARY, activeTab: 4 }
  },
  computed: {
    roles() { return this.user.roles || [] }
  },
  methods: {
    maskPhone(p) {
      if (!p || p.length < 7) return p
      return p.slice(0, 3) + ' **** ' + p.slice(-4)
    },
    onMenu(name) { this.$toast(name) }
  }
}
</script>

<style scoped>
.my-page {
  min-height: 100vh;
  background: #F5F5F7;
  padding-bottom: 50px;
}
.profile-header {
  background: linear-gradient(160deg, #F19871, #E8865A, #DC7545);
  padding: 30px 20px 28px;
}
.company-name {
  font-size: 26px; font-weight: 800; color: #fff;
  letter-spacing: 1px; margin-bottom: 18px;
}
.header-row {
  display: flex; align-items: flex-start; gap: 14px;
}
.avatar-box {
  width: 60px; height: 60px; border-radius: 16px;
  background: rgba(255,255,255,0.22);
  border: 2px solid rgba(255,255,255,0.35);
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0; overflow: hidden; cursor: pointer;
}
.avatar-img { width: 100%; height: 100%; object-fit: cover; }
.avatar-text { font-size: 26px; font-weight: 700; color: #fff; }
.header-info { flex: 1; padding-top: 4px; }
.user-name { font-size: 18px; font-weight: 600; color: #fff; }
.user-phone { font-size: 13px; color: rgba(255,255,255,0.6); margin-top: 4px; }
.user-dept { font-size: 12px; color: rgba(255,255,255,0.5); margin-top: 2px; }

.notify-btn {
  width: 38px; height: 38px; border-radius: 50%;
  background: rgba(255,255,255,0.18);
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0; cursor: pointer; position: relative; margin-top: 2px;
}
.notify-btn ::v-deep .van-badge {
  position: absolute; top: -4px; right: -4px;
}

.tags-row {
  display: flex; flex-wrap: wrap; gap: 6px;
  margin-top: 16px;
}
.tag {
  padding: 4px 12px; border-radius: 20px;
  background: rgba(255,255,255,0.15);
  font-size: 12px; color: #fff; font-weight: 500;
}

.stats-row {
  display: flex; margin-top: 20px;
  padding-top: 18px;
  border-top: 1px solid rgba(255,255,255,0.12);
}
.stat { flex: 1; text-align: center; }
.stat-num { font-size: 20px; font-weight: 700; color: #fff; }
.stat-label { font-size: 11px; color: rgba(255,255,255,0.5); margin-top: 2px; }

.menu-section { padding: 0 12px; margin-top: 14px; }
.menu-gap { margin-top: 8px; }
::v-deep .van-cell-group { border-radius: 16px; overflow: hidden; }
::v-deep .van-cell { font-size: 15px; }
::v-deep .van-cell__right-icon { display: flex; align-items: center; gap: 4px; }
.arrow-icon { font-size: 14px; color: #C7C7CC; }

::v-deep .van-tabbar-item__icon { font-size: 22px; }
</style>
