<script setup lang="ts">
import { onMounted, computed, watch } from 'vue'
import Gameboard, { CellValue, Direction } from './GameBoard'
import { socket, state } from '../socket';

const props = defineProps({
  myBoard: {
    type: Boolean,
    default: false,
  },
  opponentBoard: {
    type: Boolean,
    default: false,
  },
})
const { myBoard, opponentBoard } = props
const gb = new Gameboard({ socket })
const gameStarted = computed(() => state.gameStarted);
const gameOver = computed(() => state.gameOver);

let mouseOnBoard = false
let lastHoveredCoords: Coordinate | null = null

onMounted(() => {
  if (myBoard) {
    window.addEventListener('keydown', (event) => {
      if (
        event.code === 'Space' &&
        event.target === document.body &&
        mouseOnBoard &&
        lastHoveredCoords &&
        !gameStarted.value
      ) {
        event.preventDefault()
        gb.resetAllHover()
        gb.changeDirection()
        onHover(lastHoveredCoords)
      }
    })
  }
})

watch(() => state.attacked, (arg) => {
  if (myBoard) gb.receiveAttack(arg.coordinate, arg.attackResult);
});

function onCellClick({ x, y }: Coordinate) {
  if (!gameOver.value) {
    if (myBoard && !gameStarted.value) gb.placeShip({ x, y })
    else if (opponentBoard && gameStarted.value) gb.sendAttack({ x, y })
  }
}

function onHover({ x, y }: Coordinate) {
  if (myBoard && !gameStarted.value && gb.curShip) {
    lastHoveredCoords = { x, y }
    const placement: Placement = {
      start: { x, y },
      shipLength: gb.curShip.len,
      direction: gb.direction,
    }
    if (gb.isValidPlacement(placement)) {
      setPlacementTo(placement, CellValue.HOVER)
    }
  }
}

function onHoverLeave({ x, y }: Coordinate) {
  if (myBoard && !gameStarted.value && gb.curShip) {
    const placement: Placement = {
      start: { x, y },
      shipLength: gb.curShip.len,
      direction: gb.direction,
    }
    if (gb.isValidPlacement(placement)) {
      setPlacementTo(placement, CellValue.EMPTY)
    }
  }
}

function setPlacementTo(placement: Placement, cellValue: CellValue) {
  const {
    start: { x, y },
    shipLength,
    direction,
  } = placement
  for (let i = 0; i < shipLength; ++i) {
    if (direction === Direction.DOWN) gb.board.value[y + i][x] = cellValue
    else gb.board.value[y][x + i] = cellValue
  }
}

function setMouseOnBoard(onBoard: boolean) {
  mouseOnBoard = onBoard;
}

</script>

<template>
  <table class="board" :class="{ myBoard, opponentBoard, gameActive: !gameOver }" @mouseover="setMouseOnBoard(true)"
    @mouseleave="setMouseOnBoard(false)">
    <tr v-for="(row, y) in gb.board.value" :key="y">
      <td v-for="(col, x) in row" :key="x">
        <div :class="{
          hovered: gb.board.value[y][x] === CellValue.HOVER,
          ship: gb.board.value[y][x] === CellValue.SHIP,
          hit: gb.board.value[y][x] === CellValue.HIT,
          miss: gb.board.value[y][x] === CellValue.MISS,
        }" @click="onCellClick({ x, y })" @mouseover="onHover({ x, y })" @mouseleave="onHoverLeave({ x, y })"
          class="cell">
        </div>
      </td>
    </tr>
  </table>
</template>

<style scoped>
.board {
  width: 100%;
  min-width: 300px;
  aspect-ratio: 1 / 1;
  border: 1px solid blue;
  border-collapse: collapse;
  font-family: monospace;
}

@media screen and (min-width: 400px) {
  .board {
    width: 400px;
  }
}

td {
  padding: 0;
  margin: 0;
  border: 1px solid black;
}

.cell {
  aspect-ratio: 1 / 1;
  height: 40px;
  font-size: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.gameActive.opponentBoard .cell:not(.miss):not(.hit):hover {
  cursor: pointer;
}

.gameActive.myBoard .cell {
  cursor: pointer;
}

.hovered {
  background-color: blue;
}

.ship {
  background-color: red;
}

.opponentBoard .hit::before {
  content: "X";
}

.opponentBoard .miss::before {
  content: "•";
}

.myBoard .hit::before {
  content: "X";
}

.myBoard .hit {
  background-color: red;
}

.myBoard .miss::before {
  content: "•";
}

.opponentHit {
  background-color: grey;
}

.opponentMiss {
  background-color: orange;
}
</style>y
