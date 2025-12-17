<template>
  <div class="app-container">
    <nav class="main-nav">
      <div class="logo">ColorMaster</div>
      <div class="links">
        <router-link to="/">Генератор</router-link>
        <router-link to="/library">Библиотека</router-link>
      </div>
    </nav>
    <router-view v-slot="{ Component }">
      <transition name="fade" mode="out-in">
        <component :is="Component" />
      </transition>
    </router-view>
  </div>
</template>

<style>
body { margin: 0; font-family: 'Inter', sans-serif; background: #f4f6f8; }
.main-nav {
  display: flex; justify-content: space-between; padding: 1rem 2rem;
  background: white; box-shadow: 0 2px 10px rgba(0,0,0,0.05);
}
.logo { font-weight: bold; font-size: 1.2rem; color: #333; }
.links a {
  margin-left: 20px; text-decoration: none; color: #666; font-weight: 500;
}
.links a.router-link-active { color: #42b883; }
.fade-enter-active, .fade-leave-active { transition: opacity 0.3s; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>