<template>
  <div class="generator">
    <div class="toolbar">
      <div class="tool-group">
        <label>Режим:</label>
        <select v-model="mode">
          <option value="random">Случайный</option>
          <option value="analogous">Аналогичный</option>
          <option value="monochrome">Монохромный</option>
          <option value="triad">Триада</option>
          <option value="complementary">Комплементарный</option>
        </select>
      </div>

      <div class="tool-group">
        <label>Кол-во:</label>
        <select v-model="paletteSize" @change="handleSizeChange">
          <option :value="3">3 цвета</option>
          <option :value="5">5 цветов</option>
          <option :value="7">7 цветов</option>
        </select>
      </div>

      <div class="tool-group" v-if="mode !== 'random'">
        <label>База:</label>
        <input type="color" v-model="baseColor" @input="generateHarmony">
      </div>
      
      <div class="actions">
        <button class="btn primary" @click="generatePalette">Генерировать</button>
        <button class="btn secondary" @click="savePalette">Сохранить</button>
        <button class="btn outline" @click="showExport = true">Экспорт</button>
      </div>
    </div>

    <div class="palette-container">
      <div 
        v-for="(color, index) in colors" 
        :key="index"
        class="color-strip"
        :style="{ backgroundColor: color.hex }"
      >
        <div class="color-details" :style="{ color: getTextColor(color.hex) }">
          <span class="hex-value">{{ color.hex }}</span>
          
          <div class="badges">
             <span class="badge" title="Уровень контрастности">
              {{ getAccessLevel(color.hex) }}
            </span>
          </div>

          <button class="lock-btn" @click="toggleLock(index)">
            {{ color.locked ? '🔒' : '🔓' }}
          </button>
        </div>
      </div>
    </div>

    <div class="preview-section">
      <h3>Предпросмотр интерфейса</h3>
      <div class="mockup-card">
        <div class="mockup-nav" :style="{ backgroundColor: colors[0]?.hex || '#ccc' }">
          <span class="mockup-logo" :style="{ color: getTextColor(colors[0]?.hex || '#ccc') }">Logo</span>
          <div class="mockup-links" :style="{ color: getTextColor(colors[0]?.hex || '#ccc') }">
            <span>Home</span><span>About</span>
          </div>
        </div>
        
        <div class="mockup-body">
          <h2 :style="{ color: colors[1]?.hex || '#333' }">Заголовок страницы</h2>
          <p>
            Это пример того, как цвета вашей палитры могут выглядеть в реальном интерфейсе. 
            Мы используем первый цвет для шапки, второй для заголовков.
          </p>
          <div class="mockup-buttons">
            <button class="mockup-btn" :style="{ backgroundColor: colors[2]?.hex || 'blue', color: getTextColor(colors[2]?.hex || 'blue') }">
              Основная кнопка
            </button>
            <button class="mockup-btn" v-if="colors[3]" :style="{ backgroundColor: colors[3]?.hex, color: getTextColor(colors[3]?.hex) }">
              Вторичная
            </button>
          </div>
        </div>
      </div>
    </div>

    <ExportModal v-if="showExport" :colors="colors" @close="showExport = false" />
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { hexToHsl, hslToHex, getContrastRatio } from '../utils/colorUtils'
import ExportModal from '../components/ExportModal.vue'

// Состояние 
const colors = ref([])
const mode = ref('random')
const baseColor = ref('#42b883')
const showExport = ref(false)
const paletteSize = ref(5)

// Методы помощники
const getTextColor = (hex) => getContrastRatio(hex) === 'black' ? '#000' : '#FFF'
const getAccessLevel = (hex) => getContrastRatio(hex) === 'white' ? 'AAA' : 'AA'

// Случайный HEX
const randomHex = () => {
  return '#' + Math.floor(Math.random()*16777215).toString(16).padStart(6, '0').toUpperCase()
}

// Обработка изменения размера 
const handleSizeChange = () => {
  generatePalette()
}

// Главная функция генерации
const generatePalette = () => {
  if (mode.value === 'random') {
    const newColors = []

    for (let i = 0; i < paletteSize.value; i++) {
      if (colors.value[i]?.locked) {
        newColors.push(colors.value[i])
      } else {
        newColors.push({ hex: randomHex(), locked: false })
      }
    }
    colors.value = newColors
  } else {
    generateHarmony()
  }
}

// Генерация Гармоний 
const generateHarmony = () => {
  const { h, s, l } = hexToHsl(baseColor.value)
  const newColors = []
  
  // 1. Базовый цвет всегда первый
  newColors.push({ hex: baseColor.value, locked: true })

  // 2. Генерируем остальные цвета в зависимости от размера
  for (let i = 1; i < paletteSize.value; i++) {
    let newH = h, newS = s, newL = l;

    if (mode.value === 'analogous') {
      // Сдвигаем на 30 градусов для каждого следующего цвета
      newH = (h + (i * 30)) % 360;
    } else if (mode.value === 'monochrome') {
      // Меняем насыщенность и светлоту
      newS = Math.max(0, s - (i * 10)); 
      newL = Math.min(100, l + (i * 10)); 
      if (i % 2 === 0) newL = Math.max(0, l - (i * 10)); 
    } else if (mode.value === 'triad') {
       const step = i % 2 === 0 ? 240 : 120;
       newH = (h + step) % 360;
       if (i > 2) newL = (l + (i * 15)) % 100; 
    } else if (mode.value === 'complementary') {
       newH = (h + 180) % 360;

       if (i > 1) {
         newH = i % 2 === 0 ? h : (h + 180) % 360; // Чередуем базу и комплимент
         newL = i > 3 ? 85 : 30; // Добавляем темные и светлые версии
       }
    }

    newColors.push({ hex: hslToHex(newH, newS, newL), locked: false })
  }
  colors.value = newColors
}

const toggleLock = (index) => {
  if (colors.value[index]) colors.value[index].locked = !colors.value[index].locked
}

// Сохранение
const savePalette = () => {
  const name = prompt('Назовите палитру:', 'Моя палитра')
  if (!name) return

  const newPalette = {
    id: Date.now(),
    name,
    colors: colors.value.map(c => c.hex),
    date: new Date().toLocaleDateString()
  }

  const existing = JSON.parse(localStorage.getItem('savedPalettes') || '[]')
  existing.push(newPalette)
  localStorage.setItem('savedPalettes', JSON.stringify(existing))
  alert('Палитра сохранена в библиотеку!')
}

// Следим за режимом, чтобы перегенерировать при смене
watch(mode, () => {
  if (mode.value !== 'random') {
    colors.value.forEach(c => c.locked = false)
  }
  generatePalette()
})

onMounted(() => {
  generatePalette()
})
</script>

<style scoped>
.generator { padding: 20px; max-width: 1200px; margin: 0 auto; padding-bottom: 100px; }

/* Тулбар */
.toolbar {
  display: flex; flex-wrap: wrap; gap: 20px; margin-bottom: 20px; align-items: center; 
  background: white; padding: 20px; border-radius: 12px; 
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
}
.tool-group { display: flex; flex-direction: column; gap: 5px; font-size: 0.9rem; font-weight: 600; color: #555; }
.tool-group select, .tool-group input {
  padding: 8px; border: 1px solid #ddd; border-radius: 6px; font-size: 1rem;
}
.actions { display: flex; gap: 10px; margin-left: auto; }

/* Палитра */
.palette-container {
  display: flex; height: 50vh; border-radius: 15px; overflow: hidden;
  box-shadow: 0 10px 25px rgba(0,0,0,0.1); margin-bottom: 40px;
}
.color-strip {
  flex: 1; display: flex; align-items: center; justify-content: center;
  transition: flex 0.3s; position: relative;
}
.color-strip:hover { flex: 1.5; }

.color-details {
  text-align: center; display: flex; flex-direction: column; gap: 10px; align-items: center;
  z-index: 2;
}
.hex-value { font-size: 1.2rem; font-weight: bold; text-transform: uppercase; letter-spacing: 1px; }
.badge {
  background: rgba(255,255,255,0.25); padding: 4px 8px; border-radius: 4px; font-size: 0.8rem; backdrop-filter: blur(4px);
}
.lock-btn { 
  background: rgba(255,255,255,0.3); border: none; font-size: 1.2rem; cursor: pointer; 
  border-radius: 50%; width: 40px; height: 40px; transition: all 0.2s;
}
.lock-btn:hover { background: rgba(255,255,255,0.6); transform: scale(1.1); }

/* Кнопки */
.btn {
  padding: 10px 20px; border: none; border-radius: 8px; cursor: pointer; font-weight: 600; transition: transform 0.1s;
}
.btn:active { transform: scale(0.98); }
.primary { background: #42b883; color: white; }
.secondary { background: #35495e; color: white; }
.outline { border: 2px solid #ddd; background: transparent; }

/* Предпросмотр (Mockup) */
.preview-section { margin-top: 40px; }
.preview-section h3 { margin-bottom: 20px; color: #666; }
.mockup-card {
  border: 1px solid #eee; border-radius: 12px; overflow: hidden; 
  box-shadow: 0 4px 15px rgba(0,0,0,0.05); background: white;
}
.mockup-nav {
  padding: 1rem 2rem; display: flex; justify-content: space-between; align-items: center;
}
.mockup-logo { font-weight: 900; font-size: 1.2rem; }
.mockup-links { display: flex; gap: 15px; font-size: 0.9rem; opacity: 0.9; }
.mockup-body { padding: 3rem; text-align: center; }
.mockup-body h2 { margin-bottom: 1rem; }
.mockup-body p { max-width: 600px; margin: 0 auto 2rem; line-height: 1.6; opacity: 0.7; }
.mockup-buttons { display: flex; gap: 10px; justify-content: center; }
.mockup-btn {
  padding: 10px 24px; border: none; border-radius: 6px; font-weight: bold; cursor: pointer;
}

/* Адаптивность */
@media (max-width: 768px) {
  .palette-container { flex-direction: column; height: auto; }
  .color-strip { height: 100px; width: 100%; }
  .toolbar { flex-direction: column; align-items: stretch; }
  .actions { flex-direction: column; margin-left: 0; }
}
</style>