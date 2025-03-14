<template>
  <div class="stroke-width">
    <input 
      type="range" 
      v-model="width" 
      min="1" 
      max="20" 
      @input="emitWidth"
    >
    <span class="label">Width: {{ width }}</span>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const width = ref(2)
const emit = defineEmits(['width-selected'])

const emitWidth = () => {
  emit('width-selected', Number(width.value))
}
</script>

<style scoped>
.stroke-width {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  width: 120px;
}

input[type="range"] {
  width: 100%;
  cursor: pointer;
  -webkit-appearance: none;
  background: transparent;
  padding: 8px 0;
}

input[type="range"]::-webkit-slider-runnable-track {
  width: 100%;
  height: 4px;
  background: #e0e0e0;
  border-radius: 2px;
  transition: background 0.2s ease;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 18px;
  width: 18px;
  border-radius: 50%;
  background: #2196f3;
  margin-top: -7px;
  box-shadow: 0 2px 4px rgba(33,150,243,0.3);
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

input[type="range"]:hover::-webkit-slider-thumb {
  transform: scale(1.1);
  box-shadow: 0 4px 8px rgba(33,150,243,0.4);
}

input[type="range"]:hover::-webkit-slider-runnable-track {
  background: #2196f3;
}

.label {
  font-size: 12px;
  font-weight: 500;
  color: #666;
  letter-spacing: 0.5px;
}
</style>
