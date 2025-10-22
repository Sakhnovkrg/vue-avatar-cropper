# Vue Avatar Cropper

A lightweight, feature-rich image cropping component for Vue 3 with intuitive pan, zoom, and crop functionality.

## Demo

[Live Demo](https://sakhnovkrg.github.io/vue-avatar-cropper/)

## Features

- ✨ Intuitive drag-to-pan interface
- 🔍 Pinch-to-zoom and mouse wheel zoom support
- 📱 Mobile-friendly with multi-touch gestures
- 👁️ Optional grid overlay and circle mask
- 🖼️ Configurable output size
- 🚀 Easy to use with programmatic API
- 🔧 Full TypeScript support
- ⚡ Lightweight with no dependencies

## Installation

```bash
npm install @sakhnovkrg/vue-avatar-cropper
```

## Basic Usage

```vue
<script setup lang="ts">
import { ref } from 'vue'
import AvatarCropper from '@sakhnovkrg/vue-avatar-cropper'

const cropperRef = ref<InstanceType<typeof AvatarCropper>>()

const handleFileSelect = (event: Event) => {
  const target = event.target as HTMLInputElement
  const file = target.files?.[0]
  if (file && cropperRef.value) {
    cropperRef.value.loadImage(file)
  }
}

const handleCropped = (base64: string) => {
  console.log('Cropped image:', base64)
  // Use the base64 image as needed
}
</script>

<template>
  <div>
    <input type="file" accept="image/*" @change="handleFileSelect" />

    <AvatarCropper
      ref="cropperRef"
      :output-size="512"
      :show-grid="true"
      :show-circle="false"
      background-color="#ffffff"
      @cropped="handleCropped"
    />

    <button @click="cropperRef?.cropImage()">Crop Image</button>
  </div>
</template>
```

## Props

| Prop              | Type      | Default     | Description                                    |
| ----------------- | --------- | ----------- | ---------------------------------------------- |
| `outputSize`      | `number`  | `512`       | Output image size in pixels (width and height) |
| `showCircle`      | `boolean` | `false`     | Show circular crop overlay                     |
| `showGrid`        | `boolean` | `false`     | Show grid lines overlay                        |
| `backgroundColor` | `string`  | `'#ffffff'` | Background color of the crop area              |

## Events

| Event     | Payload          | Description                                               |
| --------- | ---------------- | --------------------------------------------------------- |
| `cropped` | `base64: string` | Emitted when image is cropped, returns base64 encoded PNG |

## Methods (via ref)

Access these methods through the component reference:

| Method      | Parameters   | Description                              |
| ----------- | ------------ | ---------------------------------------- |
| `loadImage` | `file: File` | Load an image file into the cropper      |
| `zoomIn`    | -            | Zoom in by preset step                   |
| `zoomOut`   | -            | Zoom out by preset step                  |
| `resetZoom` | -            | Reset zoom and position to initial state |
| `cropImage` | -            | Crop the image and emit the result       |
