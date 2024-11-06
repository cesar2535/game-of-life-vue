<script setup lang="ts">
import { reactive, useTemplateRef } from "vue";
import Cell from "./Cell.vue";

export type CellProps = {
  x: number;
  y: number;
};

type Props = {
  width: number;
  height: number;
  cells: CellProps[];
  cellSize: number;
};

const props = defineProps<Props>();
const emit = defineEmits<{ click: [cell: CellProps] }>();
const boardStyle = reactive({
  width: `${props.width}px`,
  height: `${props.height}px`,
});

const boardRef = useTemplateRef("board");

function getElementOffset() {
  const boardElement = boardRef.value!;
  const rect = boardElement.getBoundingClientRect();
  const doc = document.documentElement;

  return {
    x: rect.left + window.scrollX - doc.clientLeft,
    y: rect.top + window.scrollY - doc.scrollTop,
  };
}

function handleClick(event: MouseEvent) {
  const offsets = getElementOffset();
  const offsetX = event.clientX - offsets.x;
  const offsetY = event.clientY - offsets.y;

  const x = Math.floor(offsetX / props.cellSize);
  const y = Math.floor(offsetY / props.cellSize);

  const cols = props.width / props.cellSize;
  const rows = props.height / props.cellSize;

  if (x >= 0 && x <= cols && y >= 0 && y <= rows) {
    emit("click", { x, y });
  }
}
</script>

<template>
  <div ref="board" class="board" :style="boardStyle" @click="handleClick">
    <Cell
      v-for="cell in cells"
      :key="cell.x + '-' + cell.y"
      :x="cell.x"
      :y="cell.y"
      :size="cellSize"
    />
  </div>
</template>

<style lang="css" scoped>
.board {
  position: relative;
  margin: 0 auto;
  background-color: #000;
  background-image: linear-gradient(#333 1px, transparent 1px),
    linear-gradient(90deg, #333 1px, transparent 1px);
}
</style>
