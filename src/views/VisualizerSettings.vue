<template>
  <div
    class="panel"
    v-show="open"
  >
    <h3>Visualization Settings</h3>

    <label>
      Colour
      <input type="color" v-model="local.colour" />
    </label>

    <label>
      Bars
      <input
        type="range"
        min="16"
        max="512"
        step="16"
        v-model.number="local.barCount"
      />
    </label>

    <label>
      Line Width
      <input
        type="range"
        min="1"
        max="10"
        v-model.number="local.lineWidth"
      />
    </label>

    <label>
      Smoothing
      <input
        type="range"
        min="0"
        max="0.95"
        step="0.05"
        v-model.number="local.smoothing"
      />
    </label>

    <div class="buttons">
      <button @click="local.mode = 'line'">Line</button>
      <button @click="local.mode = 'bars'">Bars</button>
      <button @click="local.mode = 'circle'">Circle</button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, watch } from "vue";
import type { VisualSettings } from "@/types/visualSettings";

const props = defineProps<{
  modelValue: VisualSettings;
  open: boolean;
}>();

const emit = defineEmits<{
  (e: "update:modelValue", v: VisualSettings): void;
}>();

const local = reactive({ ...props.modelValue });

watch(
  () => local,
  () => emit("update:modelValue", { ...local }),
  { deep: true }
);
</script>

<style scoped>
.panel {
  position: fixed;
  right: 0;
  top: 0;
  width: 70%;
  height: 100%;
  padding: 16px;
  background: rgba(0, 0, 0, 0.55);
  color: #fff;
  backdrop-filter: blur(8px);
  z-index: 10;
}

label {
  display: block;
  margin-bottom: 12px;
}

.buttons{
  display: flex;
  align-items: center;
  justify-content: center;
}

.buttons button {
  margin: 5px;
  width: 30vw;
  height: 5vh;
}
</style>
