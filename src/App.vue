<script setup lang="ts">
import { times } from "lodash";
import { reactive, ref } from "vue";
import Board, { CellProps } from "./components/Board.vue";

const DIRS = [
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
];

const state = reactive<{
  width: number;
  height: number;
  cellSize: 20;
  cells: CellProps[];
  interval: number;
  isRunning: boolean;
}>({
  width: 800,
  height: 600,
  cellSize: 20,
  cells: [],
  interval: 100,
  isRunning: false,
});

const table = reactive({
  cols: state.width / state.cellSize,
  rows: state.height / state.cellSize,
});

function initialize() {
  const cols = state.width / state.cellSize;
  const rows = state.height / state.cellSize;
  const board = times(rows, () => times(cols, () => false));
  return board;
}

function calculateNeighbors(board: boolean[][], x: number, y: number) {
  const neighbors = DIRS.reduce<number>((acc, dir) => {
    const y1 = y + dir[0];
    const x1 = x + dir[1];
    const hasNeighbors =
      x1 >= 0 && x1 < table.cols && y1 >= 0 && y1 < table.rows && board[y1][x1];

    return hasNeighbors ? acc + 1 : acc;
  }, 0);

  return neighbors;
}

const board = ref(initialize());
const frame = reactive({
  then: 0,
  startTime: 0,
  id: -1,
});

function makeCells() {
  return board.value.reduce<CellProps[]>((acc, rows, y) => {
    const cells = rows.reduce<CellProps[]>((acc, item, x) => {
      return item ? acc.concat({ x, y }) : acc;
    }, []);
    return acc.concat(cells);
  }, []);
}

function loop() {
  frame.id = window.requestAnimationFrame(loop);

  const now = Date.now();
  const elasped = now - frame.then;

  if (elasped > state.interval) {
    frame.then = now - (elasped % state.interval);
    const tempBoard = initialize();

    const nextBoard = tempBoard.map((row, y) => {
      return row.map((col, x) => {
        const neighbors = calculateNeighbors(board.value, x, y);
        if (board.value[y][x]) {
          if (neighbors === 2 || neighbors === 3) {
            return true;
          } else {
            return false;
          }
        } else {
          if (!board.value[y][x] && neighbors === 3) {
            return true;
          }

          return col;
        }
      });
    });

    board.value = nextBoard;
    state.cells = makeCells();
  }
}

function stopLoop() {
  if (frame.id) {
    window.cancelAnimationFrame(frame.id);
    frame.id = -1;
  }
}

function handleStart() {
  state.isRunning = true;

  frame.then = Date.now();
  frame.startTime = frame.then;
  frame.id = window.requestAnimationFrame(loop);
}

function handleStop() {
  state.isRunning = false;
  stopLoop();
}

function handleClick(cell: CellProps) {
  board.value[cell.y][cell.x] = !board.value[cell.y][cell.x];
  state.cells = makeCells();
}

function handleRandom() {
  const nextBoard = board.value.map((row) =>
    row.map(() => Math.random() >= 0.5)
  );

  board.value = nextBoard;
  state.cells = makeCells();
}

function handleClear() {
  board.value = initialize();
  state.cells = makeCells();
  handleStop();
}
</script>

<template>
  <div class="app">
    <div>Total Cells: {{ state.cells.length }}</div>
    <Board
      :width="state.width"
      :height="state.height"
      :cell-size="state.cellSize"
      :cells="state.cells"
      @click="handleClick"
    />
    <div class="controls">
      Update every
      <input v-model="state.interval" /> ms
      <button v-if="state.isRunning" class="button" @click="handleStop">
        Stop
      </button>
      <button v-else class="button" @click="handleStart">Run</button>
      <button class="button" @click="handleRandom">Random</button>
      <button class="button" @click="handleClear">Clear</button>
    </div>
  </div>
</template>

<style scoped>
.app {
  text-align: center;
}

.controls {
  margin-top: 20px;
}

.controls input {
  font-size: 16px;
  text-align: right;
  width: 100px;
  padding: 5px 10px;
}

.button {
  font-size: 16px;
  padding: 5px 10px;
  border-radius: 3px;
  margin-left: 12px;
  cursor: pointer;
}
</style>
