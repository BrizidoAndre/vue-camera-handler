<script setup>
import {onMounted, reactive, useTemplateRef, watch} from "vue";
import './assets/bootstrap.min.css'
import Range from "@/components/Range.vue";

const videoRef = useTemplateRef('videoRef');
const imageRef = useTemplateRef('imageRef');
const canvasRef = useTemplateRef('canvasRef');
let stream;
let ctx;
const images = reactive({
  blur: 0,
  grayscale: 0,
  contrast: 100,
  opacity: 1,
  turn: false,
})

async function startCamera() {
  try {
    stopTracks();
    stream = await navigator.mediaDevices.getUserMedia({
      audio: false,
      video: {
        facingMode: 'user',
      }
    });

    videoRef.value.srcObject = stream;
    videoRef.value.onloadedmetadata = () => {
      canvasRef.value.width = videoRef.value.videoWidth;
      canvasRef.value.height = videoRef.value.videoHeight;
      videoRef.value.play();
    }
  } catch (e) {
    console.warn('It is not possible to start the camera')
  }
}

function stopTracks() {
  if (stream) {
    stream.getTracks().forEach(track => track.stop());
    stream = null;
  }
}

function printImage() {
  ctx.drawImage(videoRef.value, 0, 0, canvasRef.value.width, canvasRef.value.height);
  imageRef.value.src = canvasRef.value.toDataURL();
}

function render() {
  ctx.drawImage(imageRef.value, 0, 0, canvasRef.value.width, canvasRef.value.height);
}

function downloadImage() {
  if (!imageRef.value.src) {
    alert('Não há imagem para download');
    return null;
  }
  const link = document.createElement('a');
  link.download = 'donwload-images';
  link.href = canvasRef.value.toDataURL();
  link.click();
}

onMounted(() => {
  ctx = canvasRef.value.getContext('2d');
})

watch(images, value => {
  ctx.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height);
  ctx.filter = `blur(${value.blur}px) grayscale(${value.grayscale}%) contrast(${value.contrast}%) opacity(${value.opacity})`;
  render()
})

function turnImage() {
  ctx.translate(canvasRef.value.width, 0);
  ctx.scale(-1, 1);
  render();
}
</script>

<template>
  <div class="container row g-1">
    <div class="col-6">
      <video ref="videoRef"></video>
      <img ref="imageRef" alt="" style="display: none">
      <div>
        <button class="btn btn-primary" @click="startCamera">Start camera</button>
        <button class="btn btn-secondary" @click="printImage">Print image</button>
        <button class="btn btn-success" @click="downloadImage">Download image</button>
      </div>
      <div>
        <range min="0" max="15" v-model="images.blur">Blur</range>
        <range min="0" max="100" v-model="images.grayscale">Grayscale</range>
        <range min="0" max="200" v-model="images.contrast">Contrast</range>
        <range min="0" max="1" step=".05" v-model="images.opacity">Opacity</range>
        <button class="btn btn-outline-secondary" @click="turnImage">Turn</button>
      </div>
    </div>
    <div class="col-6">
      <canvas ref="canvasRef"></canvas>
    </div>
  </div>
</template>