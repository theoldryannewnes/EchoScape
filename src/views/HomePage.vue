<template>
  <div class="container">
    <canvas ref="canvas"></canvas>

    <button v-if="!started" class="start-button" @click="startAudio">
      Enable Audio
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from "vue";

const canvas = ref<HTMLCanvasElement | null>(null);

const audioContext = ref<AudioContext | null>(null);
const analyser = ref<AnalyserNode | null>(null);
const microphone = ref<MediaStreamAudioSourceNode | null>(null);
const dataArray = ref<Uint8Array | null>(null);
const started = ref<boolean>(false);

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
  analyser.value.fftSize = 2048;

  dataArray.value = new Uint8Array(analyser.value.fftSize);

  microphone.value.connect(analyser.value);

  draw();
}

function draw(): void {
  requestAnimationFrame(draw);

  if (
    !canvas.value ||
    !analyser.value ||
    !dataArray.value
  ) return;

  const ctx = canvas.value.getContext("2d");
  if (!ctx) return;

  analyser.value.getByteTimeDomainData(
    dataArray.value as Uint8Array<ArrayBuffer>
  );

  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);

  ctx.lineWidth = 2;
  ctx.strokeStyle = "lime";
  ctx.beginPath();

  const slice = canvas.value.width / dataArray.value.length;
  let x = 0;

  for (let i = 0; i < dataArray.value.length; i++) {
    const v = (dataArray.value[i] - 128) / 128;
    const y =
      canvas.value.height / 2 +
      v * (canvas.value.height / 3);

    if (i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);

    x += slice;
  }

  ctx.stroke();
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
