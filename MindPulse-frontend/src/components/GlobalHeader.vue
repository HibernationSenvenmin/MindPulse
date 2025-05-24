<template>
  <a-row id="globalHeader" align="center" :wrap="false">
    <a-col flex="auto">
      <a-menu
        mode="horizontal"
        :selected-keys="selectedKeys"
        @menu-item-click="doMenuClick"
      >
        <a-menu-item
          key="0"
          :style="{ padding: 0, marginRight: '38px' }"
          disabled
        >
          <div class="titleBar">
            <img class="logo" src="../assets/logo.png" />
            <div class="title">MindPulse</div>
          </div>
        </a-menu-item>
        <a-menu-item v-for="item in visibleRoutes" :key="item.path">
          {{ item.name }}
        </a-menu-item>
      </a-menu>
    </a-col>
    <a-col flex="100px">
      <div v-if="loginUserStore.loginUser.id">
        <a-dropdown trigger="hover" @select="handleSelect">
          <a-button type="primary" shape="round">{{
            loginUserStore.loginUser.userName ?? "无名"
          }}</a-button>
          <template #content>
            <a-doption @click="handleLogout">退出登录</a-doption>
          </template>
        </a-dropdown>
      </div>
      <div v-else>
        <a-button type="primary" href="/user/login">登录</a-button>
      </div>
    </a-col>
  </a-row>
</template>

<script setup lang="ts">
import { routes } from "@/router/routes";
import { useRouter } from "vue-router";
import { computed, ref } from "vue";
import { useLoginUserStore } from "@/store/userStore";
import checkAccess from "@/access/checkAccess";
import { Message } from "@arco-design/web-vue";
import { userLogoutUsingPost } from "@/api/userController";

const loginUserStore = useLoginUserStore();

const router = useRouter();
// 当前选中的菜单项
const selectedKeys = ref(["/"]);
// 路由跳转时，自动更新选中的菜单项
router.afterEach((to, from, failure) => {
  selectedKeys.value = [to.path];
});

// 展示在菜单栏的路由数组
const visibleRoutes = computed(() => {
  return routes.filter((item) => {
    if (item.meta?.hideInMenu) {
      return false;
    }
    // 根据权限过滤菜单
    if (!checkAccess(loginUserStore.loginUser, item.meta?.access as string)) {
      return false;
    }
    return true;
  });
});

// 点击菜单跳转到对应页面
const doMenuClick = (key: string) => {
  router.push({
    path: key,
  });
};

// 退出登录

const handleLogout = async () => {
  const res = await userLogoutUsingPost();
  if (res.data.code === 0) {
    // 1. 清除用户存储的登录信息
    loginUserStore.setLoginUser({
      id: undefined,
      userName: undefined,
    });

    // 2. 清除本地存储的token（如果有）
    localStorage.removeItem("token");
    sessionStorage.removeItem("token");

    // 3. 跳转到首页
    await router.push({
      path: "/",
      query: {
        redirect: router.currentRoute.value.path,
      },
    });

    // 4. 显示退出成功消息
    Message.success("已安全退出");
  } else {
    Message.error("退出登录失败");
    console.error("退出登录错误");
  }
};
</script>

<style scoped>
#globalHeader {
}

.titleBar {
  display: flex;
  align-items: center;
}

.title {
  margin-left: 16px;
  color: black;
}

.logo {
  height: 48px;
}
</style>
