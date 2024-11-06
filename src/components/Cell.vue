<script setup lang="ts">
import { reactive } from "vue";

type Props = {
  x: number;
  y: number;
  size: number;
};

const props = defineProps<Props>();
const styleObject = reactive({
  top: `${props.size * props.y + 1}px`,
  left: `${props.size * props.x + 1}px`,
  width: `${props.size - 1}px`,
  height: `${props.size - 1}px`,
});
</script>

<template>
  <div class="cell" :style="styleObject" />
</template>

<style lang="css" scoped>
.cell {
  background-color: #ccc;
  position: absolute;
}
</style>
