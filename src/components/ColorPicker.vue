<template>
  <div class="color-picker">
    <input type="color" v-model="selectedColor" @input="emitColor">
    <span class="label">Color</span>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const selectedColor = ref('#000000')
const emit = defineEmits(['color-selected'])

const emitColor = () => {
  emit('color-selected', selectedColor.value)
}
</script>

<style scoped>
.color-picker {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

input[type="color"] {
  width: 48px;
  height: 48px;
  padding: 0;
  border: 2px solid #eaeaea;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

input[type="color"]:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
}

.label {
  font-size: 12px;
  font-weight: 500;
  color: #666;
  letter-spacing: 0.5px;
}
</style>
