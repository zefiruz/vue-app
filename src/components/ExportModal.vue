<template>
  <div class="modal-backdrop" @click="$emit('close')">
    <div class="modal-content" @click.stop>
      <h3>Экспорт CSS</h3>
      <textarea readonly :value="cssCode"></textarea>
      <div class="modal-actions">
        <button @click="copyCode">Копировать</button>
        <button @click="$emit('close')">Закрыть</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
const props = defineProps(['colors'])

const cssCode = computed(() => {
  let css = ':root {\n'
  props.colors.forEach((c, i) => {
    css += `  --color-${i + 1}: ${c.hex};\n`
  })
  css += '}'
  return css
})

const copyCode = () => {
  navigator.clipboard.writeText(cssCode.value)
  alert('CSS скопирован!')
}
</script>

<style scoped>
.modal-backdrop {
  position: fixed; top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0,0,0,0.5); display: flex; align-items: center; justify-content: center;
  z-index: 1000;
}
.modal-content {
  background: white; padding: 20px; border-radius: 12px; width: 400px;
}
textarea { width: 100%; height: 150px; margin: 10px 0; font-family: monospace; }
.modal-actions { display: flex; justify-content: flex-end; gap: 10px; }
</style>