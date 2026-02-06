<template>
  <div class="container" @mousedown="onPressStart" @mouseup="onPressEnd" @touchstart="onPressStart" @touchend="onPressEnd">
    <canvas ref="canvas"></canvas>

    <button v-if="!started" class="start-button" @click="startAudio">
      Enable Audio
    </button>

    <VisualizerSettings v-model="settings" :open="settingsOpen" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";

import VisualizerSettings from "./VisualizerSettings.vue";
import type { VisualSettings } from "@/types/visualSettings";


const canvas = ref<HTMLCanvasElement | null>(null);

const audioContext = ref<AudioContext | null>(null);
const analyser = ref<AnalyserNode | null>(null);
const microphone = ref<MediaStreamAudioSourceNode | null>(null);
const dataArray = ref<Uint8Array | null>(null);
const started = ref<boolean>(false);

const settings = ref<VisualSettings>({
  colour: "#00ff88",
  barCount: 128,
  mode: "line",
  lineWidth: 2,
  smoothing: 0.9
});

const settingsOpen = ref<boolean>(false);

function resize(): void {
  if (!canvas.value) return;
  canvas.value.width = window.innerWidth;
  canvas.value.height = window.innerHeight;
}

onMounted(() => {
  resize();
  window.addEventListener("resize", resize);
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", resize);
});

let pressTimer: number | null = null;

function onPressStart(): void {
  pressTimer = window.setTimeout(() => {
    settingsOpen.value = !settingsOpen.value;
  }, 600);
}

function onPressEnd(): void {
  if (pressTimer !== null) {
    clearTimeout(pressTimer);
    pressTimer = null;
  }
}

async function startAudio(): Promise<void> {
  if (started.value) return;
  started.value = true;

  audioContext.value = new AudioContext();
  await audioContext.value.resume();

  const stream: MediaStream =
    await navigator.mediaDevices.getUserMedia({ audio: true });

  microphone.value =
    audioContext.value.createMediaStreamSource(stream);

  analyser.value = audioContext.value.createAnalyser();
  // analyser.value.fftSize = 2048;
  analyser.value.fftSize = 1024;

  dataArray.value = new Uint8Array(analyser.value.fftSize);

  microphone.value.connect(analyser.value);

  draw();
}

function draw(): void {
  requestAnimationFrame(draw);

  if (!canvas.value || !analyser.value || !dataArray.value)
    return;

  const ctx = canvas.value.getContext("2d");
  if (!ctx)
    return;

  const mode = settings.value.mode;

  analyser.value.smoothingTimeConstant = settings.value.smoothing;

  if (mode === "line") {
    analyser.value.getByteTimeDomainData(
      dataArray.value as Uint8Array<ArrayBuffer>
    );
  } else {
    analyser.value.getByteFrequencyData(
      dataArray.value as Uint8Array<ArrayBuffer>
    );
  }

  //Clear the canvas before drawing
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);

  if (mode === "line") {
    ctx.lineWidth = settings.value.lineWidth;
    ctx.strokeStyle = settings.value.colour;
    ctx.beginPath();

    const bars = settings.value.barCount;
    const step = Math.floor(dataArray.value.length / bars);
    const slice = canvas.value.width / bars;
    let x = 0;

    for (let i = 0; i < bars; i++) {
      const v =
        (dataArray.value[i * step] - 128) / 128;

      const y =
        canvas.value.height / 2 +
        v * (canvas.value.height / 3);

      if (i === 0) ctx.moveTo(x, y);
      else ctx.lineTo(x, y);

      x += slice;
    }

    ctx.stroke();
  } if (mode === "bars") {
    const bars = settings.value.barCount;
    const halfBars = Math.floor(bars / 2);
    const step = Math.floor(dataArray.value.length / halfBars);

    const centerX = canvas.value.width / 2;
    const barWidth = canvas.value.width / bars;

    ctx.fillStyle = settings.value.colour;

    for (let i = 0; i < halfBars; i++) {
      const value = dataArray.value[i * step] / 255;
      const barHeight = value * canvas.value.height;

      const xRight =
        centerX + i * barWidth;
      const xLeft =
        centerX - (i + 1) * barWidth;

      const y =
        canvas.value.height - barHeight;

      ctx.fillRect(
        xRight,
        y,
        barWidth - 1,
        barHeight
      );

      ctx.fillRect(
        xLeft,
        y,
        barWidth - 1,
        barHeight
      );
    }
} if (mode === "circle") {
    const cx = canvas.value.width / 2;
    const cy = canvas.value.height / 2;
    const baseRadius = Math.min(cx, cy) * 0.35;

    const bars = settings.value.barCount;
    const halfBars = Math.floor(bars / 2);
    const step = Math.floor(dataArray.value.length / halfBars);

    ctx.strokeStyle = settings.value.colour;
    ctx.lineWidth = settings.value.lineWidth;

    for (let i = 0; i < halfBars; i++) {
        const value = dataArray.value[i * step] / 255;
        const magnitude = value * baseRadius;

        const angleA = (i / halfBars) * Math.PI;
        const angleB = angleA + Math.PI;

        const x1 =
          cx + Math.cos(angleA) * baseRadius;
        const y1 =
          cy + Math.sin(angleA) * baseRadius;

        const x2 =
          cx + Math.cos(angleA) * (baseRadius + magnitude);
        const y2 =
          cy + Math.sin(angleA) * (baseRadius + magnitude);

        const x3 =
          cx + Math.cos(angleB) * baseRadius;
        const y3 =
          cy + Math.sin(angleB) * baseRadius;

        const x4 =
          cx + Math.cos(angleB) * (baseRadius + magnitude);
        const y4 =
          cy + Math.sin(angleB) * (baseRadius + magnitude);

        ctx.beginPath();
        ctx.moveTo(x1, y1);
        ctx.lineTo(x2, y2);
        ctx.stroke();

        ctx.beginPath();
        ctx.moveTo(x3, y3);
        ctx.lineTo(x4, y4);
        ctx.stroke();
      }
    }

}
</script>

<style scoped>
.container {
  width: 100%;
  height: 100%;
  position: relative;
}

canvas {
  display: block;
  width: 100%;
  height: 100%;
}

.start-button {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 16px 32px;
  font-size: 18px;
  background: #00ff88;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}
</style>
