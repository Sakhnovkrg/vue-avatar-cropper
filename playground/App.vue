<script setup lang="ts">
import { ref } from 'vue'
import AvatarCropper from '../src/AvatarCropper.vue'

const OUTPUT_SIZE = 512

const cropperRef = ref<InstanceType<typeof AvatarCropper>>()
const showCircle = ref(false)
const showGrid = ref(true)
const fileInput = ref<HTMLInputElement>()

const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (file && cropperRef.value) {
    cropperRef.value.loadImage(file)
  }
}

const handleCropped = (base64: string) => {
  fetch(base64)
    .then(res => res.blob())
    .then(blob => {
      const url = URL.createObjectURL(blob)
      window.open(url, '_blank')
    })
}
</script>

<template>
  <div class="app">
    <h1 class="title">Vue Avatar Cropper</h1>

    <div class="controls">
      <input
        ref="fileInput"
        type="file"
        accept="image/*"
        style="display: none"
        @change="handleFileSelect"
      />
      <button class="button" @click="fileInput?.click()">Select Photo</button>
      <button class="button" @click="cropperRef?.zoomIn()">+</button>
      <button class="button" @click="cropperRef?.zoomOut()">-</button>
      <button class="button" @click="cropperRef?.resetZoom()">Reset</button>
      <button class="button button--success" @click="cropperRef?.cropImage()">
        Crop
      </button>
      <label class="checkbox">
        <input v-model="showGrid" type="checkbox" />
        Grid
      </label>
      <label class="checkbox">
        <input v-model="showCircle" type="checkbox" />
        Circle
      </label>
    </div>

    <div class="cropper">
      <AvatarCropper
        ref="cropperRef"
        :output-size="OUTPUT_SIZE"
        :show-circle="showCircle"
        :show-grid="showGrid"
        background-color="#1a1a1a"
        @cropped="handleCropped"
      />
    </div>
  </div>
</template>

<style>
*, *::after, *::before {
  box-sizing: border-box;
}

:root {
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    sans-serif;
}

body {
  margin: 0;
}

.app {
  min-height: 100vh;
  padding: 20px;
  max-width: 100%;
}

.title {
  text-align: center;
  margin-bottom: 20px;
  color: #42b883;
}

.controls {
  display: flex;
  gap: 8px;
  margin-bottom: 20px;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  padding: 0 10px;
}

.button {
  padding: 10px 16px;
  background: #646cff;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: background 0.3s;
  white-space: nowrap;
}

.button:hover {
  background: #535bf2;
}

.button:active {
  transform: scale(0.98);
}

.button--success {
  background: #42b883;
}

.button--success:hover {
  background: #33a372;
}

.checkbox {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-size: 14px;
  white-space: nowrap;
}

.checkbox input[type='checkbox'] {
  cursor: pointer;
}

.cropper {
  max-width: 600px;
  width: 100%;
  margin: 0 auto;
}

@media (max-width: 768px) {
  .controls {
    gap: 6px;
  }

  .button {
    padding: 8px 12px;
    font-size: 13px;
  }

  .checkbox {
    font-size: 13px;
  }

  .cropper {
    max-width: 100%;
  }
}

@media (max-width: 480px) {
  .title {
    font-size: 24px;
  }

  .controls {
    gap: 5px;
  }

  .button {
    padding: 6px 10px;
    font-size: 12px;
  }

  .checkbox {
    font-size: 12px;
    width: 100%;
    justify-content: center;
  }
}
</style>
