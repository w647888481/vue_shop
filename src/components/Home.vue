<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div class="avatar">
        <img src="../assets/user.jpg" alt="" />
        <span>电商后台管理系统</span>
      </div>
      <el-button round @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleCollapse">
          <i class="el-icon-s-fold el-icon" v-if="isCollapse === false"></i>
          <i class="el-icon-s-unfold el-icon" v-else></i>
        </div>
        <!-- 侧边栏菜单区域 -->
        <el-menu background-color="#5c7dff" text-color="#fff" active-text-color="#ff9900" unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconsObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/' + subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/' + subItem.path)">
              <template slot="title">
                <!-- 图标 -->
                <i class="el-icon-menu"></i>
                <!-- 文本 -->
                <span>{{ subItem.authName }}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      //左侧菜单栏数据
      menulist: [],
      iconsObj: {
        125: 'iconfont icon-user',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao',
      },
      isCollapse: false,
      activePath: '',
    }
  },
  created() {
    this.getMenuList(), (this.activePath = window.sessionStorage.getItem('activePath'))
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
      this.$message.success('退出成功！')
    },
    //获取所有菜单
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menulist = res.data
    },
    //点击按钮，切换菜单折叠与展开
    toggleCollapse() {
      this.isCollapse = !this.isCollapse
    },
    //保存链接的激活状态
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    },
  },
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
  background-color: #f1f1f1;
}
.el-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: auto;
  color: #fff;
  font-size: 150%;
  font-weight: 700;
  background: linear-gradient(90deg, #348ac7, #7474bf);
  > div {
    display: flex;
    align-items: center;
    > img {
      width: 50px;
      height: 50px;
      border-radius: 25px;
    }
    > span {
      padding-left: 10px;
    }
  }
}

.el-aside {
  background-color: #5c7dff;
  .el-menu {
    border-right: none;
  }
}

.el-main {
  background-color: #f1f1f1;
  padding: 0;
}

.iconfont {
  margin-right: 10px;
}

.toggle-button {
  background-color: #5375fa;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}

.el-icon {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24px;
}
</style>
