<script setup lang="ts">
import { ref } from 'vue'
import AvatarCropper from '../src/AvatarCropper.vue'

const OUTPUT_SIZE = 512

const cropperRef = ref<InstanceType<typeof AvatarCropper>>()
const showCircle = ref(false)
const showGrid = ref(true)
const fileInput = ref<HTMLInputElement>()
const showModal = ref(false)
const croppedImage = ref('')

const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (file && cropperRef.value) {
    cropperRef.value.loadImage(file)
  }
}

const handleCropped = (base64: string) => {
  croppedImage.value = base64
  showModal.value = true
}

const closeModal = () => {
  showModal.value = false
}

const downloadImage = () => {
  const link = document.createElement('a')
  link.href = croppedImage.value
  link.download = 'cropped-avatar.png'
  link.click()
}
</script>

<template>
  <div class="app">
    <h1 class="title">Vue Avatar Cropper</h1>
    <p class="subtitle">
      <a
        href="https://github.com/sakhnovkrg/vue-avatar-cropper"
        target="_blank"
        rel="noopener noreferrer"
      >
        View on GitHub
      </a>
    </p>

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

    <div v-if="showModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <button class="modal-close" @click="closeModal">&times;</button>
        <h2 class="modal-title">Cropped Image</h2>
        <img :src="croppedImage" alt="Cropped" class="modal-image" />
        <div class="modal-actions">
          <button class="button button--success" @click="downloadImage">
            Download
          </button>
          <button class="button" @click="closeModal">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
*,
*::after,
*::before {
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
  margin-bottom: 8px;
  color: #42b883;
}

.subtitle {
  text-align: center;
  margin-top: 0;
  margin-bottom: 20px;
}

.subtitle a {
  color: #646cff;
  text-decoration: none;
  font-size: 14px;
  transition: color 0.3s;
}

.subtitle a:hover {
  color: #535bf2;
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

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
}

.modal-content {
  background: white;
  border-radius: 12px;
  padding: 30px;
  max-width: 600px;
  width: 100%;
  position: relative;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-close {
  position: absolute;
  top: 10px;
  right: 15px;
  background: none;
  border: none;
  font-size: 32px;
  cursor: pointer;
  color: #666;
  line-height: 1;
  padding: 0;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.3s;
}

.modal-close:hover {
  color: #333;
}

.modal-title {
  margin: 0 0 20px 0;
  color: #333;
  text-align: center;
}

.modal-image {
  width: 100%;
  height: auto;
  border-radius: 8px;
  margin-bottom: 20px;
  display: block;
}

.modal-actions {
  display: flex;
  gap: 10px;
  justify-content: center;
}

@media (max-width: 480px) {
  .modal-content {
    padding: 20px;
  }

  .modal-title {
    font-size: 20px;
  }

  .modal-actions {
    flex-direction: column;
  }

  .modal-actions .button {
    width: 100%;
  }
}
</style>
