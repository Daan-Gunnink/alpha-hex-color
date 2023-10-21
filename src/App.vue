<template>
  <div class="container">
    <h1>Alpha Color Picker</h1>
    <div class="color-wrapper">
      <div>
        <h2>RGB</h2>
        <h2>{{ rgb }}</h2>
      </div>
      <div>
        <h2>RGBA</h2>
        <h2>{{ rgba }}</h2>
      </div>
      <div>
        <h2>HSLA</h2>
        <h2>{{ hsla }}</h2>
      </div>
    </div>
    <div class="picker-wrapper">
      <color-picker v-bind="color" @input="onInput"></color-picker>
      <div class="slider-wrapper">
        <input v-model="color.hue" />
        <input v-model="color.saturation" />
        <input v-model="color.luminosity" />
        <input v-model="color.alpha" />
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, watch } from "vue";
import ColorPicker from "@radial-color-picker/vue-color-picker";

const color = reactive({
  hue: 50,
  saturation: 100,
  luminosity: 50,
  alpha: 1,
});

const onInput = (hue) => {
  color.hue = hue;
};

const HSLToRGB = (h, s, l) => {
  s /= 100;
  l /= 100;
  const k = (n) => (n + h / 30) % 12;
  const a = s * Math.min(l, 1 - l);
  const f = (n) =>
    l - a * Math.max(-1, Math.min(k(n) - 3, Math.min(9 - k(n), 1)));
  return {
    r: 255 * f(0),
    g: 255 * f(4),
    b: 255 * f(8),
  };
};

function toRGB() {
  const {r,g,b} = HSLToRGB(color.hue, color.saturation, color.luminosity);
  return `rgb(${Math.round(r)},${Math.round(g)},${Math.round(b)})`
}

function toRGBA() {
  const {r,g,b} = HSLToRGB(color.hue, color.saturation, color.luminosity);
  return `rgb(${Math.round(r)},${Math.round(g)},${Math.round(b)},${color.alpha})`
}

function toHSLA() {
  return `hsla(${color.hue},${color.saturation}%,${color.luminosity}%,${color.alpha})`;
}

const hsla = ref(toHSLA());
const rgb = ref(toRGB());
const rgba = ref(toRGBA());

watch(color, () => {
  hsla.value = toHSLA();
  rgb.value = toRGB();
  rgba.value = toRGBA();
});
</script>

<style>
@import "@radial-color-picker/vue-color-picker/dist/vue-color-picker.min.css";

.container {
  height: 100vh;
  justify-content: center;
}

.picker-wrapper {
  gap: 2em;
  display: flex;
  flex-direction: row;
  justify-content: center;
}

.slider-wrapper {
  gap: 1em;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.color-wrapper {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 10em;
}
</style>
