<script setup lang="ts">
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://vuejs.org/api/sfc-script-setup.html#script-setup
import Greet from "./components/Greet.vue"
import { Camera } from "@mediapipe/camera_utils"
import { ControlPanel, DropDownControl, FPS, FireFn, ControlPanelListener, Slider, SourcePicker, SourceType, StaticText, Toggle } from "@mediapipe/control_utils"
import { clamp, drawConnectors, drawLandmarks, drawRectangle, lerp } from "@mediapipe/drawing_utils"
import { Holistic, POSE_CONNECTIONS, FACEMESH_TESSELATION, HAND_CONNECTIONS } from "@mediapipe/holistic"
import { ref, onMounted } from "vue"

const videoElement = ref<HTMLVideoElement>()
const canvasElement = ref<HTMLCanvasElement>()

onMounted(() => {

  const canv = canvasElement.value as HTMLCanvasElement
  const canvasCtx = canv.getContext('2d') as CanvasRenderingContext2D;

  function onResults(results: any) {
    canvasCtx.save();

    const width = canvasElement.value?.width as number
    const height = canvasElement.value?.height as number
    canvasCtx.clearRect(0, 0, width, height);
    canvasCtx.drawImage(results.segmentationMask, 0, 0, width, height);

    // Only overwrite existing pixels.
    canvasCtx.globalCompositeOperation = 'source-in';
    canvasCtx.fillStyle = '#00FF00';
    canvasCtx.fillRect(0, 0, width, height);

    // Only overwrite missing pixels.
    canvasCtx.globalCompositeOperation = 'destination-atop';
    canvasCtx.drawImage(
      results.image, 0, 0, width, height);

    canvasCtx.globalCompositeOperation = 'source-over';
    drawConnectors(canvasCtx, results.poseLandmarks, POSE_CONNECTIONS,
      { color: '#00FF00', lineWidth: 4 });
    drawLandmarks(canvasCtx, results.poseLandmarks,
      { color: '#FF0000', lineWidth: 2 });
    drawConnectors(canvasCtx, results.faceLandmarks, FACEMESH_TESSELATION,
      { color: '#C0C0C070', lineWidth: 1 });
    drawConnectors(canvasCtx, results.leftHandLandmarks, HAND_CONNECTIONS,
      { color: '#CC0000', lineWidth: 5 });
    drawLandmarks(canvasCtx, results.leftHandLandmarks,
      { color: '#00FF00', lineWidth: 2 });
    drawConnectors(canvasCtx, results.rightHandLandmarks, HAND_CONNECTIONS,
      { color: '#00CC00', lineWidth: 5 });
    drawLandmarks(canvasCtx, results.rightHandLandmarks,
      { color: '#FF0000', lineWidth: 2 });
    canvasCtx.restore();
  }

  const holistic = new Holistic({
    locateFile: (file) => {
      return `/@mediapipe/holistic/${file}`;
    }
  });
  holistic.setOptions({
    modelComplexity: 1,
    smoothLandmarks: true,
    enableSegmentation: true,
    smoothSegmentation: true,
    refineFaceLandmarks: true,
    minDetectionConfidence: 0.5,
    minTrackingConfidence: 0.5
  });
  holistic.onResults(onResults);

  const camera = new Camera(videoElement.value as HTMLVideoElement, {
    onFrame: async () => {
      await holistic.send({ image: videoElement.value as HTMLVideoElement });
    },
    width: 1280,
    height: 720
  });
  camera.start();
})

</script>

<template>
  <div class="container">

    <video class="input_video" ref="videoElement"></video>
    <canvas class="output_canvas" ref="canvasElement" :width=1280 :height=720></canvas>

    <h1>Welcome to Tauri!</h1>

    <div class="row">
      <a href="https://vitejs.dev" target="_blank">
        <img src="/vite.svg" class="logo vite" alt="Vite logo" />
      </a>
      <a href="https://tauri.app" target="_blank">
        <img src="/tauri.svg" class="logo tauri" alt="Tauri logo" />
      </a>
      <a href="https://vuejs.org/" target="_blank">
        <img src="./assets/vue.svg" class="logo vue" alt="Vue logo" />
      </a>
    </div>

    <p>Click on the Tauri, Vite, and Vue logos to learn more.</p>

    <p>
      Recommended IDE setup:
      <a href="https://code.visualstudio.com/" target="_blank">VS Code</a>
      +
      <a href="https://github.com/johnsoncodehk/volar" target="_blank">Volar</a>
      +
      <a href="https://github.com/tauri-apps/tauri-vscode" target="_blank">Tauri</a>
      +
      <a href="https://github.com/rust-lang/rust-analyzer" target="_blank">rust-analyzer</a>
    </p>

    <Greet />
  </div>
</template>

<style scoped>
.logo.vite:hover {
  filter: drop-shadow(0 0 2em #747bff);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #249b73);
}
</style>
