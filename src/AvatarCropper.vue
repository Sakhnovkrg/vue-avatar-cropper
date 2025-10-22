<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

const MIN_SCALE = 0.1
const MAX_SCALE = 10
const ZOOM_STEP = 1.2
const ZOOM_SPEED = 0.1

interface Props {
  outputSize?: number
  showCircle?: boolean
  showGrid?: boolean
  backgroundColor?: string
}

const props = withDefaults(defineProps<Props>(), {
  outputSize: 512,
  showCircle: false,
  showGrid: false,
  backgroundColor: '#ffffff',
})

const emit = defineEmits<{
  cropped: [base64: string]
}>()

const canvas = ref<HTMLCanvasElement>()
const container = ref<HTMLDivElement>()
const image = ref<HTMLImageElement>()

const imageLoaded = ref(false)
const imageUrl = ref<string>('')
const scale = ref(1)
const position = ref({ x: 0, y: 0 })
const isDragging = ref(false)
const dragStart = ref({ x: 0, y: 0 })

const initialDistance = ref(0)
const lastScale = ref(1)

const cropSize = computed(() => {
  if (!container.value) return 0
  const width = container.value.clientWidth
  const height = container.value.clientHeight
  return Math.min(width, height)
})

const imageStyle = computed(() => {
  if (!image.value) return {}

  const scaledWidth = image.value.naturalWidth * scale.value
  const scaledHeight = image.value.naturalHeight * scale.value
  const translateX = (position.value.x / 100) * scaledWidth
  const translateY = (position.value.y / 100) * scaledHeight

  return {
    transform: `translate(${translateX}px, ${translateY}px) scale(${scale.value})`,
    transformOrigin: '0 0',
  }
})

const loadImage = (file: File) => {
  if (file && file.type.startsWith('image/')) {
    if (imageUrl.value) {
      URL.revokeObjectURL(imageUrl.value)
    }

    imageUrl.value = URL.createObjectURL(file)
    imageLoaded.value = false
  }
}

const onImageLoad = () => {
  if (!image.value) return

  const img = image.value
  const currentCropSize = cropSize.value

  if (currentCropSize <= 0) return

  const scaleX = currentCropSize / img.naturalWidth
  const scaleY = currentCropSize / img.naturalHeight
  scale.value = Math.max(scaleX, scaleY)
  lastScale.value = scale.value

  const scaledWidth = img.naturalWidth * scale.value
  const scaledHeight = img.naturalHeight * scale.value

  position.value = {
    x: (((currentCropSize - scaledWidth) / scaledWidth) * 100) / 2,
    y: (((currentCropSize - scaledHeight) / scaledHeight) * 100) / 2,
  }

  imageLoaded.value = true
}

const getDistance = (touch1: Touch, touch2: Touch): number => {
  const dx = touch1.clientX - touch2.clientX
  const dy = touch1.clientY - touch2.clientY
  return Math.sqrt(dx * dx + dy * dy)
}

const applyZoomFromCenter = (newScale: number) => {
  if (!image.value) return

  const img = image.value
  const currentCropSize = cropSize.value
  const oldScaledWidth = img.naturalWidth * scale.value
  const oldScaledHeight = img.naturalHeight * scale.value
  const newScaledWidth = img.naturalWidth * newScale
  const newScaledHeight = img.naturalHeight * newScale

  const centerOffsetX =
    currentCropSize / 2 - (position.value.x / 100) * oldScaledWidth
  const centerOffsetY =
    currentCropSize / 2 - (position.value.y / 100) * oldScaledHeight

  position.value = {
    x:
      ((currentCropSize / 2 - (centerOffsetX * newScale) / scale.value) /
        newScaledWidth) *
      100,
    y:
      ((currentCropSize / 2 - (centerOffsetY * newScale) / scale.value) /
        newScaledHeight) *
      100,
  }

  scale.value = newScale
}

const onMouseDown = (event: MouseEvent) => {
  if (!imageLoaded.value || !image.value) return

  isDragging.value = true
  dragStart.value = {
    x: event.clientX,
    y: event.clientY,
  }
}

const handleDrag = (clientX: number, clientY: number) => {
  if (!image.value) return

  const img = image.value
  const scaledWidth = img.naturalWidth * scale.value
  const scaledHeight = img.naturalHeight * scale.value

  const deltaX = clientX - dragStart.value.x
  const deltaY = clientY - dragStart.value.y

  position.value = {
    x: position.value.x + (deltaX / scaledWidth) * 100,
    y: position.value.y + (deltaY / scaledHeight) * 100,
  }

  dragStart.value = { x: clientX, y: clientY }
}

const onMouseMove = (event: MouseEvent) => {
  if (!isDragging.value) return
  handleDrag(event.clientX, event.clientY)
}

const onMouseUp = () => {
  isDragging.value = false
}

const onTouchStart = (event: TouchEvent) => {
  if (!imageLoaded.value || !image.value) return

  if (event.touches.length === 1) {
    const touch = event.touches[0]
    if (!touch) return

    isDragging.value = true
    dragStart.value = {
      x: touch.clientX,
      y: touch.clientY,
    }
  } else if (event.touches.length === 2) {
    const touch0 = event.touches[0]
    const touch1 = event.touches[1]
    if (!touch0 || !touch1) return

    isDragging.value = false
    initialDistance.value = getDistance(touch0, touch1)
    lastScale.value = scale.value
  }
}

const onTouchMove = (event: TouchEvent) => {
  if (!imageLoaded.value || !image.value) return

  event.preventDefault()

  if (event.touches.length === 1 && isDragging.value) {
    const touch = event.touches[0]
    if (!touch) return

    handleDrag(touch.clientX, touch.clientY)
  } else if (event.touches.length === 2) {
    const touch0 = event.touches[0]
    const touch1 = event.touches[1]
    if (!touch0 || !touch1) return

    const currentDistance = getDistance(touch0, touch1)
    const scaleChange = currentDistance / initialDistance.value
    const newScale = Math.max(
      MIN_SCALE,
      Math.min(MAX_SCALE, lastScale.value * scaleChange)
    )

    applyZoomFromCenter(newScale)
  }
}

const onTouchEnd = () => {
  isDragging.value = false
  lastScale.value = scale.value
}

const onWheel = (event: WheelEvent) => {
  if (!imageLoaded.value || !image.value) return

  event.preventDefault()

  const delta = -Math.sign(event.deltaY) * ZOOM_SPEED
  const newScale = Math.max(
    MIN_SCALE,
    Math.min(MAX_SCALE, scale.value * (1 + delta))
  )

  applyZoomFromCenter(newScale)
}

const zoomIn = () => {
  if (!image.value) return
  const newScale = Math.min(MAX_SCALE, scale.value * ZOOM_STEP)
  applyZoomFromCenter(newScale)
}

const zoomOut = () => {
  if (!image.value) return
  const newScale = Math.max(MIN_SCALE, scale.value / ZOOM_STEP)
  applyZoomFromCenter(newScale)
}

const resetZoom = () => {
  if (!image.value) return

  const img = image.value
  const currentCropSize = cropSize.value
  const scaleX = currentCropSize / img.naturalWidth
  const scaleY = currentCropSize / img.naturalHeight
  scale.value = Math.max(scaleX, scaleY)

  const scaledWidth = img.naturalWidth * scale.value
  const scaledHeight = img.naturalHeight * scale.value

  position.value = {
    x: (((currentCropSize - scaledWidth) / scaledWidth) * 100) / 2,
    y: (((currentCropSize - scaledHeight) / scaledHeight) * 100) / 2,
  }
}

const cropImage = (): string => {
  if (!image.value || !canvas.value || !container.value) return ''

  const canvasEl = canvas.value
  const img = image.value
  const ctx = canvasEl.getContext('2d')
  if (!ctx) return ''

  canvasEl.width = props.outputSize
  canvasEl.height = props.outputSize

  const scaledWidth = img.naturalWidth * scale.value
  const scaledHeight = img.naturalHeight * scale.value

  const imgLeft = (position.value.x / 100) * scaledWidth
  const imgTop = (position.value.y / 100) * scaledHeight

  const sourceX = -imgLeft / scale.value
  const sourceY = -imgTop / scale.value
  const sourceSize = cropSize.value / scale.value

  ctx.fillStyle = 'white'
  ctx.fillRect(0, 0, props.outputSize, props.outputSize)

  ctx.drawImage(
    img,
    sourceX,
    sourceY,
    sourceSize,
    sourceSize,
    0,
    0,
    props.outputSize,
    props.outputSize
  )

  const base64 = canvasEl.toDataURL('image/png')
  emit('cropped', base64)

  return base64
}

onMounted(() => {
  document.addEventListener('mousemove', onMouseMove)
  document.addEventListener('mouseup', onMouseUp)
})

onUnmounted(() => {
  document.removeEventListener('mousemove', onMouseMove)
  document.removeEventListener('mouseup', onMouseUp)

  if (imageUrl.value) {
    URL.revokeObjectURL(imageUrl.value)
  }
})

defineExpose({
  loadImage,
  zoomIn,
  zoomOut,
  resetZoom,
  cropImage,
})
</script>

<template>
  <div class="avatar-cropper">
    <div ref="container" class="avatar-cropper__container">
      <div
        v-if="imageUrl"
        class="avatar-cropper__area"
        :style="{
          backgroundColor: backgroundColor,
        }"
        @mousedown="onMouseDown"
        @touchstart="onTouchStart"
        @touchmove="onTouchMove"
        @touchend="onTouchEnd"
        @wheel="onWheel"
      >
        <img
          ref="image"
          :src="imageUrl"
          :style="imageStyle"
          class="avatar-cropper__image"
          draggable="false"
          @load="onImageLoad"
        />

        <div class="avatar-cropper__overlay">
          <svg
            v-if="showGrid"
            class="avatar-cropper__grid"
            width="100%"
            height="100%"
          >
            <line
              x1="33.33%"
              y1="0"
              x2="33.33%"
              y2="100%"
              stroke="rgba(255, 255, 255, 0.5)"
              stroke-width="1"
            />
            <line
              x1="66.66%"
              y1="0"
              x2="66.66%"
              y2="100%"
              stroke="rgba(255, 255, 255, 0.5)"
              stroke-width="1"
            />
            <line
              x1="0"
              y1="33.33%"
              x2="100%"
              y2="33.33%"
              stroke="rgba(255, 255, 255, 0.5)"
              stroke-width="1"
            />
            <line
              x1="0"
              y1="66.66%"
              x2="100%"
              y2="66.66%"
              stroke="rgba(255, 255, 255, 0.5)"
              stroke-width="1"
            />
          </svg>

          <svg
            v-if="showCircle"
            class="avatar-cropper__circle"
            width="100%"
            height="100%"
          >
            <defs>
              <mask id="circleMask">
                <rect width="100%" height="100%" fill="white" />
                <circle cx="50%" cy="50%" r="50%" fill="black" />
              </mask>
            </defs>
            <rect
              width="100%"
              height="100%"
              fill="rgba(0, 0, 0, 0.35)"
              mask="url(#circleMask)"
            />
            <circle
              cx="50%"
              cy="50%"
              r="calc(50% - 1px)"
              fill="none"
              stroke="white"
              stroke-width="2"
              stroke-dasharray="5,5"
            />
          </svg>
        </div>
      </div>
    </div>

    <canvas ref="canvas" style="display: none"></canvas>
  </div>
</template>

<style scoped>
.avatar-cropper {
  width: 100%;
}

.avatar-cropper__container {
  position: relative;
  width: 100%;
  padding-bottom: 100%;
}

.avatar-cropper__area {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  cursor: move;
  touch-action: none;
}

.avatar-cropper__image {
  position: absolute;
  top: 0;
  left: 0;
  user-select: none;
  pointer-events: none;
  max-width: none;
}

.avatar-cropper__overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
}

.avatar-cropper__grid {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.avatar-cropper__circle {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}
</style>
