<template>
  <div class="generator-container">
    
    <div class="controls-card">
      <div class="controls-row">
        <div class="control-group">
          <label>Количество цветов:</label>
          <select v-model="paletteSize" @change="handleSizeChange" class="select-input">
            <option :value="3">3 цвета</option>
            <option :value="5">5 цветов</option>
            <option :value="7">7 цветов</option>
          </select>
        </div>

        <div class="control-group">
          <label>Формат:</label>
          <button @click="toggleFormat" class="btn-toggle">
            {{ colorFormat }}
          </button>
        </div>

        <button @click="generatePalette" class="btn-generate">
          🔄 Сгенерировать палитру
        </button>
      </div>
    </div>

    <div class="palette-display">
      <div 
        v-for="(color, index) in colors" 
        :key="index"
        class="color-card"
        :style="{ backgroundColor: color.hex }"
      >
        <button 
          class="lock-btn" 
          @click.stop="toggleLock(index)"
          :class="{ active: color.locked }"
          title="Закрепить цвет"
        >
          <span v-if="color.locked">🔒</span>
          <span v-else>🔓</span>
        </button>

        <div class="color-info" @click="copyColor(color.hex)">
          <span class="color-text" :class="{ 'light-text': isDarkColor(color.hex) }">
            {{ formatColor(color.hex) }}
          </span>
          <span class="copy-hint" :class="{ 'light-text': isDarkColor(color.hex) }">
            Скопировать
          </span>
        </div>
      </div>
    </div>

    <div v-if="notification" class="notification">
      {{ notification }}
    </div>

    <hr class="divider">

    <div class="preview-section" :class="{ 'dark-mode': isPreviewDark }">
      <div class="preview-header">
        <h2>Предпросмотр интерфейса</h2>
        <label class="switch-label">
          <input type="checkbox" v-model="isPreviewDark">
          Тёмный фон превью
        </label>
      </div>

      <div class="mockup-card" :style="mockupStyles">
        <div class="mockup-nav" :style="{ backgroundColor: colors[0]?.hex || '#ccc' }">
          <span class="mockup-logo">Logo</span>
          <div class="mockup-links">
            <span>Home</span>
            <span>About</span>
          </div>
        </div>
        
        <div class="mockup-body">
          <h3 :style="{ color: colors[1]?.hex || '#333' }">Заголовок страницы</h3>
          <p>
            Это пример того, как цвета вашей палитры могут выглядеть в реальном интерфейсе. 
            Мы используем первый цвет для шапки, второй для заголовков, а третий для акцентов.
          </p>
          <button class="mockup-btn" :style="{ backgroundColor: colors[2]?.hex || 'blue' }">
            Действие (Color 3)
          </button>
          
          <div class="mockup-tags" v-if="colors.length > 3">
            <span 
              v-for="i in (colors.length - 3)" 
              :key="i" 
              class="mockup-tag"
              :style="{ borderColor: colors[i+2]?.hex, color: colors[i+2]?.hex }"
            >
              Tag {{ i }}
            </span>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const paletteSize = ref(5) 
const colorFormat = ref('HEX') 
const notification = ref('') 
const isPreviewDark = ref(false) 

const colors = ref([])


const generateRandomHex = () => {
  const letters = '0123456789ABCDEF'
  let color = '#'
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)]
  }
  return color
}

const generatePalette = () => {
  const newColors = []
  
  for (let i = 0; i < paletteSize.value; i++) {
    if (colors.value[i] && colors.value[i].locked) {
      newColors.push(colors.value[i])
    } else {
      newColors.push({
        hex: generateRandomHex(),
        locked: false
      })
    }
  }
  
  colors.value = newColors
}

const handleSizeChange = () => {
  if (colors.value.length < paletteSize.value) {
    const diff = paletteSize.value - colors.value.length
    for (let i = 0; i < diff; i++) {
      colors.value.push({ hex: generateRandomHex(), locked: false })
    }
  } else if (colors.value.length > paletteSize.value) {
    colors.value = colors.value.slice(0, paletteSize.value)
  }
}

const toggleLock = (index) => {
  colors.value[index].locked = !colors.value[index].locked
}

const hexToRgb = (hex) => {
  const r = parseInt(hex.slice(1, 3), 16)
  const g = parseInt(hex.slice(3, 5), 16)
  const b = parseInt(hex.slice(5, 7), 16)
  return `rgb(${r}, ${g}, ${b})`
}

const formatColor = (hex) => {
  return colorFormat.value === 'HEX' ? hex : hexToRgb(hex)
}

const toggleFormat = () => {
  colorFormat.value = colorFormat.value === 'HEX' ? 'RGB' : 'HEX'
}

const copyColor = (hex) => {
  const textToCopy = formatColor(hex)
  navigator.clipboard.writeText(textToCopy).then(() => {
    notification.value = `Скопировано: ${textToCopy} ✅`
    setTimeout(() => {
      notification.value = ''
    }, 2000)
  })
}

const isDarkColor = (hex) => {
  const r = parseInt(hex.slice(1, 3), 16)
  const g = parseInt(hex.slice(3, 5), 16)
  const b = parseInt(hex.slice(5, 7), 16)

  return (r * 0.299 + g * 0.587 + b * 0.114) < 128
}

// LocalStorage 
const saveColors = () => {
  localStorage.setItem('my-palette', JSON.stringify(colors.value))
  localStorage.setItem('palette-size', paletteSize.value)
}

const loadColors = () => {
  const savedColors = localStorage.getItem('my-palette')
  const savedSize = localStorage.getItem('palette-size')
  
  if (savedSize) paletteSize.value = Number(savedSize)
  
  if (savedColors) {
    colors.value = JSON.parse(savedColors)
  } else {
    generatePalette()
  }
}

// Следим за изменениями цветов чтобы сохранять их
watch(colors, saveColors, { deep: true })
watch(paletteSize, saveColors)

onMounted(() => {
  loadColors()
})

const mockupStyles = computed(() => {
  return {
    backgroundColor: isPreviewDark.value ? '#1a1a1a' : '#ffffff',
    color: isPreviewDark.value ? '#ffffff' : '#333333'
  }
})
</script>

<style scoped>

.generator-container {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

/* Панель управления */
.controls-card {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
}

.controls-row {
  display: flex;
  flex-wrap: wrap;
  gap: 1.5rem;
  align-items: flex-end;
  justify-content: center;
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.control-group label {
  font-weight: 600;
  font-size: 0.9rem;
  color: #555;
}

.select-input, .btn-toggle, .btn-generate {
  padding: 0.6rem 1rem;
  border-radius: 8px;
  border: 1px solid #ddd;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-toggle {
  background-color: #f8f9fa;
  min-width: 80px;
}

.btn-generate {
  background-color: #42b883;
  color: white;
  border: none;
  font-weight: bold;
}

.btn-generate:hover {
  background-color: #3aa876;
  transform: translateY(-2px);
}

/* Палитра цветов */
.palette-display {
  display: flex;
  height: 200px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 10px 15px rgba(0,0,0,0.1);
}

.color-card {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
  transition: flex 0.3s ease;
  position: relative;
}

.color-card:hover {
  flex: 1.2; /* Эффект расширения при наведении */
}

/* Кнопка замка */
.lock-btn {
  background: rgba(255, 255, 255, 0.3);
  border: none;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  cursor: pointer;
  font-size: 1.2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.2s;
}

.lock-btn:hover {
  background: rgba(255, 255, 255, 0.6);
}

.lock-btn.active {
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 0 10px rgba(0,0,0,0.2);
}

/* Инфо о цвете */
.color-info {
  text-align: center;
  cursor: pointer;
  background: rgba(0, 0, 0, 0.1);
  padding: 0.5rem 1rem;
  border-radius: 20px;
  transition: transform 0.2s;
}

.color-info:hover {
  transform: scale(1.05);
}

.color-text {
  font-family: monospace;
  font-weight: bold;
  font-size: 1.1rem;
  display: block;
}

.copy-hint {
  font-size: 0.7rem;
  opacity: 0;
  display: block;
  margin-top: 2px;
}

.color-info:hover .copy-hint {
  opacity: 1;
}

/* Утилита для светлого текста на темном фоне */
.light-text {
  color: white;
  text-shadow: 0 1px 2px rgba(0,0,0,0.3);
}

/* Уведомление */
.notification {
  position: fixed;
  bottom: 30px;
  left: 50%;
  transform: translateX(-50%);
  background-color: #333;
  color: white;
  padding: 10px 20px;
  border-radius: 30px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
  animation: slideUp 0.3s ease-out;
  z-index: 100;
}

@keyframes slideUp {
  from { transform: translate(-50%, 20px); opacity: 0; }
  to { transform: translate(-50%, 0); opacity: 1; }
}

/* Разделитель */
.divider {
  border: 0;
  height: 1px;
  background: #ddd;
  margin: 1rem 0;
}

/* Превью */
.preview-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.switch-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  user-select: none;
}

.mockup-card {
  border: 1px solid #eee;
  border-radius: 8px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
  transition: background-color 0.3s, color 0.3s;
}

.mockup-nav {
  padding: 1rem 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white; /* Предполагаем, что навбар чаще всего контрастный */
}

.mockup-logo {
  font-weight: 900;
  font-size: 1.2rem;
}

.mockup-links {
  display: flex;
  gap: 15px;
  font-size: 0.9rem;
}

.mockup-body {
  padding: 2rem;
  text-align: center;
}

.mockup-body p {
  max-width: 600px;
  margin: 1rem auto;
  line-height: 1.6;
  opacity: 0.8;
}

.mockup-btn {
  margin-top: 1rem;
  padding: 10px 24px;
  border: none;
  border-radius: 6px;
  color: white;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}

.mockup-tags {
  margin-top: 2rem;
  display: flex;
  gap: 10px;
  justify-content: center;
}

.mockup-tag {
  border: 1px solid;
  padding: 4px 12px;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: 600;
}

/* Адаптивность для мобильных */
@media (max-width: 768px) {
  .palette-display {
    flex-direction: column;
    height: auto;
  }
  
  .color-card {
    height: 80px;
    flex-direction: row;
    padding: 0 20px;
  }

  .color-card:hover {
    flex: 1; /* Отключаем расширение на мобильных */
  }
}
</style>