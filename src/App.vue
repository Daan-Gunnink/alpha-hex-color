<template>
  <div class="container">
    <h1>Alpha Color Picker</h1>
    <div class="picker-wrapper">
      <div class="colors-wrapper">
        <div class="color-col">
          <span>HEX</span>
          <span>HEXA</span>
          <span>RGB</span>
          <span>RGBA</span>
          <span>HSL</span>
          <span>HSLA</span>
        </div>
        <div class="color-col">
          <span class="copyable" @click="copyToClipboard(hex)">{{ hex }}</span>
          <span class="copyable" @click="copyToClipboard(hexa)">{{
            hexa
          }}</span>
          <span class="copyable" @click="copyToClipboard(rgb)">{{ rgb }}</span>
          <span class="copyable" @click="copyToClipboard(rgba)">{{
            rgba
          }}</span>
          <span class="copyable" @click="copyToClipboard(hsla)">{{
            hsla
          }}</span>
          <span class="copyable" @click="copyToClipboard(hsla)">{{
            hsla
          }}</span>
        </div>
      </div>
      <color-picker
        class="picker"
        v-bind="color"
        @input="onInput"
      ></color-picker>
      <div class="slider-wrapper">
        <input
          class="slider hue-slider"
          type="range"
          min="0"
          max="360"
          v-model="color.hue"
        />
        <input
          class="slider saturation-slider"
          style=""
          type="range"
          min="0"
          max="100"
          :style="saturationSliderBackground"
          v-model="color.saturation"
        />
        <input
          class="slider luminosity-slider"
          type="range"
          min="0"
          max="100"
          :style="luminositySliderBackground"
          v-model="color.luminosity"
        />
        <input
          class="slider alpha-slider"
          type="range"
          min="0"
          max="100"
          :style="alphaSliderBackground"
          :value="color.alpha * 100"
          @input="(event) => (color.alpha = event.target.value / 100)"
        />
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

const hsl = ref(toHSL());
const hsla = ref(toHSLA());
const rgb = ref(toRGB());
const rgba = ref(toRGBA());
const hex = ref(toHEX());
const hexa = ref(toHEX(true));
const saturationGradient = ref(toSaturationGradientString())
const luminosityGradient = ref(toLuminosityGradientString())
const alphaGradient = ref(toAlphaGradientString());


const saturationSliderBackground = reactive({
  background: saturationGradient,
});

const luminositySliderBackground = reactive({
  background: luminosityGradient,
});

const alphaSliderBackground = reactive({
  background: alphaGradient,
});

const onInput = (hue) => {
  color.hue = hue;
};

function HSLToRGB(h, s, l) {
  s /= 100;
  l /= 100;
  const k = (n) => (n + h / 30) % 12;
  const a = s * Math.min(l, 1 - l);
  const f = (n) =>
    l - a * Math.max(-1, Math.min(k(n) - 3, Math.min(9 - k(n), 1)));
  return {
    r: Math.round(255 * f(0)),
    g: Math.round(255 * f(8)),
    b: Math.round(255 * f(4)),
  };
}

function copyToClipboard(value) {
  navigator.clipboard.writeText(value);
}

function toRGB() {
  const { r, g, b } = HSLToRGB(color.hue, color.saturation, color.luminosity);
  return `rgb(${r},${g},${b})`;
}

function toRGBA() {
  const { r, g, b } = HSLToRGB(color.hue, color.saturation, color.luminosity);
  return `rgb(${r},${g},${b},${color.alpha})`;
}

function toHSL() {
  return `hsla(${Math.round(color.hue)},${color.saturation}%,${
    color.luminosity
  })`;
}

function toHSLA() {
  return `hsla(${Math.round(color.hue)},${color.saturation}%,${
    color.luminosity
  }%,${color.alpha})`;
}

function toHEX(includeAlpha = false) {
  const { r, g, b } = HSLToRGB(color.hue, color.saturation, color.luminosity);
  const hexR = r.toString(16).padStart(2, "0");
  const hexG = g.toString(16).padStart(2, "0");
  const hexB = b.toString(16).padStart(2, "0");

  let hexA = "";
  if (includeAlpha) {
    hexA = Math.round(color.alpha * 255).toString(16).padStart(2, "0");
  }

  return `#${hexR}${hexG}${hexB}${hexA}`;
}

function toSaturationGradientString() {
  const { r:r1, g:g1, b:b1 } = HSLToRGB(color.hue, 0, color.luminosity);
  const { r:r2, g:g2, b:b2 } = HSLToRGB(color.hue, 100, color.luminosity);

  return `-moz-linear-gradient(0deg, rgb(${r1},${g1},${b1}), rgb(${r2},${g2},${b2}))`;
}

function toLuminosityGradientString() {
  const { r:r1, g:g1, b:b1 } = HSLToRGB(color.hue, color.saturation, 0);
  const { r:r2, g:g2, b:b2} = HSLToRGB(color.hue, color.saturation, 100);
  return `-moz-linear-gradient(0deg, rgb(${r1},${g1},${b1},1), rgb(${r2},${g2},${b2},1))`;
}

function toAlphaGradientString() {
  const { r, g, b } = HSLToRGB(color.hue, color.saturation, color.luminosity);
  return `-moz-linear-gradient(0deg, rgba(${r},${g},${b},0), rgba(${r},${g},${b},1))`;
}


watch(color, () => {
  hsl.value = toHSL();
  hsla.value = toHSLA();
  rgb.value = toRGB();
  rgba.value = toRGBA();
  hex.value = toHEX();
  hexa.value = toHEX(true);
  saturationGradient.value = toSaturationGradientString()
  luminosityGradient.value = toLuminosityGradientString()
  alphaGradient.value = toAlphaGradientString()
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
  flex-grow: 4;
  gap: 1em;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.colors-wrapper {
  justify-content: center;
  display: flex;
  gap: 2em;
  width: 220px;
}

.picker {
  flex-grow: 4;
}

.color-wrapper {
  display: flex;
  gap: 1em;
  flex-direction: row;
}

.copyable {
  cursor: pointer;
  font-weight: bold;
}

.color-col {
  flex: 1;
  justify-content: center;
  flex-direction: column;
  display: flex;
  align-items: start;
}

.hue-slider {
  background: -moz-linear-gradient(
    0deg,
    rgb(255, 0, 0),
    rgb(255, 255, 0),
    rgb(0, 255, 0),
    rgb(0, 255, 255),
    rgb(0, 0, 255),
    rgb(255, 0, 255),
    rgb(255, 0, 0)
  );
}

.slider {
  -webkit-appearance: none;
  appearance: none;
  border-radius: 8px;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none; /* Override default look */
  appearance: none;
  width: 25px; /* Set a specific slider handle width */
  height: 25px; /* Slider handle height */
  background: #ffffff; /* Green background */
  border: none;
  border-radius: 8px;
  cursor: pointer; /* Cursor on hover */
}

.slider::-moz-range-thumb {
  width: 25px; /* Set a specific slider handle width */
  height: 25px; /* Slider handle height */
  background: #ffffff; /* Green background */
  border: none;
  border-radius: 24px;
  cursor: pointer; /* Cursor on hover */
}
</style>
