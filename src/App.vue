<template>
  <nav class="bg-[#FED839] p-6 px-16 text-lg flex items-center space-x-5">
    <router-link to="/" class="flex items-center space-x-3">
      <img :src="isActive('/') ? require('./assets/vacancies-icon-red.svg') : require('./assets/vacancies-icon.svg')" alt="">
      <div>Объявления</div>
    </router-link> 
    <router-link to="/users" class="flex items-center space-x-3">
      <img :src="isActive('/users') ? require('./assets/users-icon-red.svg') : require('./assets/users-icon.svg')" alt="">
      <div>Пользователи</div>
    </router-link>
    <router-link to="/post" class="flex items-center space-x-3">
      <img :src="isActive('/post') ? require('./assets/parsing-icon-red.svg') : require('./assets/parsing-icon.svg')" alt="">
      <div>Настройка и парсинг</div>
    </router-link>
  </nav>
  <router-view/>
</template>

<script>
export default {
  methods: {
    isActive(path) {
      return this.$route.path === path;
    }
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background: #FFFCEC;
  min-height: 100vh;
  color: #62560E;
}

nav {
  

  a {
    font-weight: bold;
    

    &.router-link-exact-active {
      color: #FF0303;
      text-decoration: underline;
    }
  }
}
</style>
