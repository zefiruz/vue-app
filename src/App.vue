<template>
  <div class="app-container">
    <header class="main-header">
      <h1>Vue Color Palette</h1>
    </header>
    
    <main>
      <PaletteGenerator />
    </main>
    
    <footer class="main-footer">
      <p>малямов Семён  2025</p>
    </footer>
  </div>
</template>

<script setup>
import PaletteGenerator from './components/PaletteGenerator.vue'
</script>

<style>
/* Глобальные сбросы и шрифты */
body {
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f0f2f5;
  color: #333;
}

.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.main-header {
  background: linear-gradient(135deg, #42b883 0%, #35495e 100%);
  color: white;
  padding: 1.5rem;
  text-align: center;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.main-header h1 {
  margin: 0;
  font-size: 2rem;
}

.main-header p {
  margin: 0.5rem 0 0;
  opacity: 0.8;
}

main {
  flex: 1;
  padding: 2rem;
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

.main-footer {
  text-align: center;
  padding: 1rem;
  color: #666;
  background-color: #fff;
  border-top: 1px solid #eee;
}
</style>