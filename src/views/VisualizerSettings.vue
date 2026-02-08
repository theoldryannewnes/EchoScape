<template>
  <div
    class="panel"
    v-show="open"
  >
    <h3 id="paneltitle">Visualization Settings</h3>

    <label>
      Colour
      <input type="color" v-model="local.colour" />
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

    <label v-if="local.mode === 'bars' || local.mode === 'circle'">
      Bars
      <input
        type="range"
        min="16"
        max="512"
        step="16"
        v-model.number="local.barCount"
      />
    </label>

    <label v-if="local.mode === 'line' || local.mode === 'circle'">
      Line Width
      <input
        type="range"
        min="1"
        max="10"
        v-model.number="local.lineWidth"
      />
    </label>

    <div class="buttons">
      <button
        :class="{ active: local.mode === 'line' }"
        @click="local.mode = 'line'"
      >
        Line
      </button>

      <button
        :class="{ active: local.mode === 'bars' }"
        @click="local.mode = 'bars'"
      >
        Bars
      </button>

      <button
        :class="{ active: local.mode === 'circle' }"
        @click="local.mode = 'circle'"
      >
        Circle
      </button>
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
  width: 60%;
  height: 100%;
  padding: 16px;
  background: rgba(0, 0, 0, 0.55);
  color: #fff;
  backdrop-filter: blur(8px);
  z-index: 10;
}

#paneltitle{
  text-align: center;
}

label {
  margin-bottom: 15px;
  display:flex;
  justify-content: space-evenly;
  align-items: center;
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

button.active {
  background: #00ff88;
  color: #000;
}
</style>
