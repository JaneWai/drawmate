<template>
  <div class="whiteboard-container">
    <div class="brand">
      <h1>Your Daily DrawMate</h1>
    </div>
    <div class="toolbar">
      <div class="toolbar-group">
        <ColorPicker @color-selected="setColor" />
        <StrokeWidth @width-selected="setStrokeWidth" />
      </div>
      
      <div class="toolbar-group">
        <button @click="setMode('select')" :class="{ active: currentMode === 'select' }" title="Select">
          <span class="material-icons">near_me</span>
          <span class="label">Select</span>
        </button>
        <button @click="setMode('draw')" :class="{ active: currentMode === 'draw' }" title="Draw">
          <span class="material-icons">brush</span>
          <span class="label">Draw</span>
        </button>
        <button @click="setMode('rect')" :class="{ active: currentMode === 'rect' }" title="Rectangle">
          <span class="material-icons">crop_square</span>
          <span class="label">Rectangle</span>
        </button>
        <button @click="setMode('circle')" :class="{ active: currentMode === 'circle' }" title="Circle">
          <span class="material-icons">circle</span>
          <span class="label">Circle</span>
        </button>
        <button @click="setMode('arrow')" :class="{ active: currentMode === 'arrow' }" title="Arrow">
          <span class="material-icons">arrow_forward</span>
          <span class="label">Arrow</span>
        </button>
        <button @click="setMode('text')" :class="{ active: currentMode === 'text' }" title="Text">
          <span class="material-icons">text_fields</span>
          <span class="label">Text</span>
        </button>
      </div>

      <div class="toolbar-group">
        <button @click="setMode('eraser')" :class="{ active: currentMode === 'eraser' }" title="Eraser">
          <span class="material-icons">auto_fix_normal</span>
          <span class="label">Eraser</span>
        </button>
        <button @click="undo" :disabled="!canUndo" title="Undo">
          <span class="material-icons">undo</span>
          <span class="label">Undo</span>
        </button>
        <button @click="clearCanvas" class="danger" title="Clear All">
          <span class="material-icons">delete_outline</span>
          <span class="label">Clear</span>
        </button>
      </div>
    </div>
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import ColorPicker from './ColorPicker.vue'
import StrokeWidth from './StrokeWidth.vue'
import { fabric } from 'fabric'

const canvas = ref<HTMLCanvasElement | null>(null)
let fabricCanvas: fabric.Canvas
const currentMode = ref('select')
const currentColor = ref('#000000')
const currentWidth = ref(2)
const canUndo = ref(false)
const undoStack = ref<string[]>([])

let isDrawing = false
let startX = 0
let startY = 0
let currentObject: any = null

const saveState = () => {
  undoStack.value.push(JSON.stringify(fabricCanvas.toJSON()))
  canUndo.value = true
}

const setMode = (mode: string) => {
  currentMode.value = mode
  fabricCanvas.isDrawingMode = mode === 'draw'
  fabricCanvas.selection = mode === 'select'
  
  if (mode === 'draw') {
    fabricCanvas.freeDrawingBrush.color = currentColor.value
    fabricCanvas.freeDrawingBrush.width = currentWidth.value
  }
  
  fabricCanvas.forEachObject(obj => {
    obj.selectable = mode === 'select'
    obj.evented = mode === 'select' || mode === 'eraser'
  })
}

const setColor = (color: string) => {
  currentColor.value = color
  if (fabricCanvas.isDrawingMode) {
    fabricCanvas.freeDrawingBrush.color = color
  }
}

const setStrokeWidth = (width: number) => {
  currentWidth.value = width
  if (fabricCanvas.isDrawingMode) {
    fabricCanvas.freeDrawingBrush.width = width
  }
}

const createArrow = (points: number[]) => {
  const [x1, y1, x2, y2] = points
  const deltaX = x2 - x1
  const deltaY = y2 - y1
  const angle = Math.atan2(deltaY, deltaX)
  const arrowLength = Math.sqrt(deltaX * deltaX + deltaY * deltaY)
  
  const headLength = Math.min(20, arrowLength / 3)
  const headAngle = 25 * Math.PI / 180
  
  const path = new fabric.Path(`
    M ${x1} ${y1}
    L ${x2} ${y2}
    M ${x2} ${y2}
    L ${x2 - headLength * Math.cos(angle - headAngle)} ${y2 - headLength * Math.sin(angle - headAngle)}
    M ${x2} ${y2}
    L ${x2 - headLength * Math.cos(angle + headAngle)} ${y2 - headLength * Math.sin(angle + headAngle)}
  `, {
    stroke: currentColor.value,
    strokeWidth: currentWidth.value,
    fill: '',
    selectable: true
  })
  
  return path
}

const handleMouseDown = (e: fabric.IEvent) => {
  if (currentMode.value === 'select' || currentMode.value === 'draw') return
  
  isDrawing = true
  const pointer = fabricCanvas.getPointer(e.e)
  startX = pointer.x
  startY = pointer.y
  
  switch (currentMode.value) {
    case 'rect':
      currentObject = new fabric.Rect({
        left: startX,
        top: startY,
        width: 0,
        height: 0,
        fill: 'transparent',
        stroke: currentColor.value,
        strokeWidth: currentWidth.value
      })
      break
      
    case 'circle':
      currentObject = new fabric.Circle({
        left: startX,
        top: startY,
        radius: 0,
        fill: 'transparent',
        stroke: currentColor.value,
        strokeWidth: currentWidth.value
      })
      break
      
    case 'text':
      currentObject = new fabric.IText('Click to type', {
        left: startX,
        top: startY,
        fontSize: currentWidth.value * 10,
        fill: currentColor.value,
        selectable: true,
        editable: true
      })
      fabricCanvas.add(currentObject)
      currentObject.enterEditing()
      currentObject.selectAll()
      isDrawing = false
      break
      
    case 'arrow':
      currentObject = createArrow([startX, startY, startX, startY])
      break
  }

  if (currentObject && currentMode.value !== 'text') {
    fabricCanvas.add(currentObject)
  }
}

const handleMouseMove = (e: fabric.IEvent) => {
  if (!isDrawing || !currentObject) return
  
  const pointer = fabricCanvas.getPointer(e.e)
  const width = pointer.x - startX
  const height = pointer.y - startY
  
  switch (currentMode.value) {
    case 'rect':
      currentObject.set({
        width: Math.abs(width),
        height: Math.abs(height),
        left: width > 0 ? startX : pointer.x,
        top: height > 0 ? startY : pointer.y
      })
      break
      
    case 'circle':
      const radius = Math.sqrt(width * width + height * height) / 2
      const centerX = startX + width / 2
      const centerY = startY + height / 2
      currentObject.set({
        radius,
        left: centerX - radius,
        top: centerY - radius
      })
      break
      
    case 'arrow':
      fabricCanvas.remove(currentObject)
      currentObject = createArrow([startX, startY, pointer.x, pointer.y])
      fabricCanvas.add(currentObject)
      break
  }
  
  fabricCanvas.renderAll()
}

const handleMouseUp = () => {
  if (isDrawing) {
    isDrawing = false
    currentObject = null
    saveState()
  }
}

const handleEraserHover = (e: fabric.IEvent) => {
  if (currentMode.value !== 'eraser') return
  
  const target = e.target
  if (target) {
    saveState()
    fabricCanvas.remove(target)
  }
}

const undo = () => {
  if (undoStack.value.length > 0) {
    const lastState = undoStack.value.pop()
    if (lastState) {
      fabricCanvas.loadFromJSON(lastState, () => {
        fabricCanvas.renderAll()
      })
    }
    canUndo.value = undoStack.value.length > 0
  }
}

const clearCanvas = () => {
  saveState()
  fabricCanvas.clear()
}

onMounted(() => {
  if (!canvas.value) return
  
  fabricCanvas = new fabric.Canvas(canvas.value, {
    width: window.innerWidth,
    height: window.innerHeight - 160,
    backgroundColor: 'white'
  })

  fabricCanvas.on('mouse:down', handleMouseDown)
  fabricCanvas.on('mouse:move', handleMouseMove)
  fabricCanvas.on('mouse:up', handleMouseUp)
  fabricCanvas.on('mouse:over', handleEraserHover)
  
  setMode('select')
  
  window.addEventListener('resize', () => {
    fabricCanvas.setWidth(window.innerWidth)
    fabricCanvas.setHeight(window.innerHeight - 160)
    fabricCanvas.renderAll()
  })
})
</script>

<style scoped>
.whiteboard-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background: #f8f9fa;
  font-family: 'Poppins', 'Segoe UI', system-ui, -apple-system, sans-serif;
}

.brand {
  background: #2196f3;
  color: white;
  padding: 12px 24px;
  text-align: center;
}

.brand h1 {
  margin: 0;
  font-size: 24px;
  font-weight: 600;
  letter-spacing: -0.5px;
}

.toolbar {
  display: flex;
  gap: 24px;
  padding: 16px 24px;
  background: white;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
  justify-content: center;
  align-items: center;
  z-index: 100;
}

.toolbar-group {
  display: flex;
  gap: 8px;
  padding: 0 16px;
  border-right: 2px solid #f0f0f0;
}

.toolbar-group:last-child {
  border-right: none;
}

button {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  padding: 8px;
  border: 2px solid #eaeaea;
  border-radius: 12px;
  background: white;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  min-width: 64px;
  color: #666;
}

button:hover {
  background: #f8f9fa;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
  border-color: #e0e0e0;
}

button.active {
  background: #2196f3;
  color: white;
  border-color: #1976d2;
  box-shadow: 0 4px 12px rgba(33,150,243,0.3);
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

button.danger {
  color: #dc3545;
  border-color: #dc3545;
}

button.danger:hover {
  background: #dc3545;
  color: white;
  box-shadow: 0 4px 12px rgba(220,53,69,0.3);
}

.material-icons {
  font-size: 24px;
}

.label {
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.5px;
}

canvas {
  flex: 1;
  background: white;
}

@media (max-width: 768px) {
  .brand h1 {
    font-size: 20px;
  }

  .toolbar {
    flex-wrap: wrap;
    gap: 16px;
  }
  
  .toolbar-group {
    flex-wrap: wrap;
    justify-content: center;
  }
  
  button {
    min-width: 56px;
  }
}
</style>
