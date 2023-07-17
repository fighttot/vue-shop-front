<template>
  <VNavigationDrawer v-if="isMobile" v-model="drawer" location="right" temporary>
    <VList nav>
      <template v-for="navItem in navItems" :key="navItem.to">
        <VListItem :to="navItem.to" v-if="navItem.show">
          <template #prepend>
            <VIcon :icon="navItem.icon"></VIcon>
          </template>
          <template #append>
            <VBadge color="success" :content="user.cart.toString()" v-if="navItem.to === '/cart'"> </VBadge>
          </template>
          <VListItemTitle>{{ navItem.text }}</VListItemTitle>

        </VListItem>
      </template>
      <VListItem v-if="user.isLogin" @click="logout">
        <template #prepend>
          <VIcon icon="mdi-logout"></VIcon>
          <VListItemTitle>登出</VListItemTitle>
        </template>
      </VListItem>

    </VList>
  </VNavigationDrawer>
  <VAppBar color="primary">
    <VContainer class="d-flex align-center">
      <VBtn to="/" variant="text" :active="false">
        <VAppBarTitle>購物網</VAppBarTitle>
      </VBtn>
      <VSpacer></VSpacer>

      <VAppBarNavIcon v-if="isMobile" @click="drawer = !drawer"></VAppBarNavIcon>
      <template v-if="!isMobile">
        <template v-for="navItem in navItems" :key="navItem.to">
          <VBtn variant="text" :prepend-icon="navItem.icon" :to="navItem.to" v-if="navItem.show"> {{ navItem.text }}
            <VBadge color="success" :content="user.cart.toString()" floating v-if="navItem.to === '/cart'"></VBadge>
          </VBtn>
        </template>
      </template>
      <VBtn variant="text" prepend-icon="mdi-logout" @click="logout" v-if="!isMobile && user.isLogin">登出</VBtn>
    </VContainer>
  </VAppBar>

  <VMain>
    <RouterView :key="$route.path">

    </RouterView>
  </VMain>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useDisplay } from 'vuetify'
import { useUserStore } from '@/store/user'
import { apiAuth } from '@/plugins/axios'
import { useSnackbar } from 'vuetify-use-dialog'
const user = useUserStore()
const { mobile } = useDisplay()
const isMobile = computed(() => mobile.value)

const createSnackbar = useSnackbar()
const drawer = ref(false)

const navItems = computed(() => {
  return [
    { to: '/register', text: '註冊', icon: 'mdi-account-plus', show: !user.isLogin },
    { to: '/login', text: '登入', icon: 'mdi-login', show: !user.isLogin },
    { to: '/cart', text: '購物車', icon: 'mdi-cart', show: user.isLogin },
    { to: '/orders', text: '訂單', icon: 'mdi-format-list-numbered', show: user.isLogin },
    { to: '/admin', text: '管理', icon: 'mdi-cog', show: user.isLogin && user.isAdmin }
  ]
})

const logout = async () => {
  try {
    await apiAuth.delete('/users/logout')
    user.logout()
    createSnackbar({
      text: '登出成功',
      showCloseButton: false,
      snackbarProps: {
        timeout: 2000,
        color: 'green',
        location: 'bottom'
      }
    })
  } catch (error) {
    createSnackbar({
      text: error.response.data.message,
      showCloseButton: false,
      snackbarProps: {
        timeout: 2000,
        color: 'red',
        location: 'bottom'
      }
    })
  }
}
</script>
