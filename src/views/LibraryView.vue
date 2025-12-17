<template>
  <div class="library">
    <h1>Ваша коллекция</h1>
    
    <div class="search-bar">
      <input v-model="search" placeholder="Поиск палитр..." />
    </div>

    <div v-if="filteredPalettes.length === 0" class="empty-state">
      Нет сохраненных палитр. Создайте новую в Генераторе!
    </div>

    <div class="grid">
      <div v-for="p in filteredPalettes" :key="p.id" class="card">
        <div class="card-preview">
          <div 
            v-for="c in p.colors" 
            :key="c" 
            class="mini-color" 
            :style="{ backgroundColor: c }"
          ></div>
        </div>
        <div class="card-info">
          <h3>{{ p.name }}</h3>
          <p>{{ p.date }}</p>
          <button class="delete-btn" @click="remove(p.id)">Удалить</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const palettes = ref([])
const search = ref('')

const load = () => {
  palettes.value = JSON.parse(localStorage.getItem('savedPalettes') || '[]')
}

const remove = (id) => {
  if(confirm('Удалить эту палитру?')) {
    palettes.value = palettes.value.filter(p => p.id !== id)
    localStorage.setItem('savedPalettes', JSON.stringify(palettes.value))
  }
}

const filteredPalettes = computed(() => {
  return palettes.value.filter(p => 
    p.name.toLowerCase().includes(search.value.toLowerCase())
  )
})

onMounted(load)
</script>

<style scoped>
.library { padding: 40px; max-width: 1200px; margin: 0 auto; }
.search-bar input {
  width: 100%; padding: 15px; font-size: 1.1rem; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 30px;
}
.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 20px; }
.card { background: white; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 10px rgba(0,0,0,0.05); }
.card-preview { display: flex; height: 120px; }
.mini-color { flex: 1; }
.card-info { padding: 15px; }
.card-info h3 { margin: 0 0 5px 0; }
.card-info p { margin: 0; color: #888; font-size: 0.9rem; }
.delete-btn {
  margin-top: 10px; background: #ff6b6b; color: white; border: none; padding: 5px 10px; border-radius: 4px; cursor: pointer;
}
</style>