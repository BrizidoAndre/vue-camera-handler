<script setup>
import {useId} from "vue";

const model = defineModel();
const id = useId();
defineOptions({
  inheritAttrs: false,
})
</script>

<template>
  <label :for="id" class="form-label">
    <slot></slot>
  </label>
  <input type="range" class="form-range" :id="id" v-bind="$attrs" v-model="model">
</template>