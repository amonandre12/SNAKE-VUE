<template>
  <div class="game-container">
    <h2>Score: {{ score }}</h2>
    <div v-for="(row, rowIndex) in grid" :key="rowIndex" class="row">
      <div v-for="(cell, cellIndex) in row" :key="cellIndex"
           :class="getCellClass(rowIndex, cellIndex)"
           class="cell">
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
// add the score
const score = ref(0)
const isPaused = ref(false)
let gameInterval = null
// Creation the Grid
const gridSize = 10;
const grid = ref(Array.from({ length: gridSize }, () => Array(gridSize).fill(null)));

 // Position for the snake

const snake = ref([{ x: 5, y: 5 }]);
const direction = ref('right');
const food = ref({ x: 3, y: 3 });

const getCellClass = (row, col) => {
  if (snake.value.some(segment => segment.x === row && segment.y === col)) {
    return 'snake';
  }
  if (food.value.x === row && food.value.y === col) {
    return 'food';
  }
  return '';
};

// Déplacer le serpent
const moveSnake = () => {
  if(isPaused.value) return

  const head = { ...snake.value[0] };

  if (direction.value === 'up') head.x--;
  if (direction.value === 'down') head.x++;
  if (direction.value === 'left') head.y--;
  if (direction.value === 'right') head.y++;

  // Vérifier si le serpent touche les murs
  if (head.x < 0 || head.x >= gridSize || head.y < 0 || head.y >= gridSize) {
    alert('Game Over ! Score : ${score.value}');
    resetGame ()
    return;
  }

  snake.value.unshift(head); // Ajouter une nouvelle tête

  // Vérifier si le serpent mange la nourriture

  if (head.x === food.value.x && head.y === food.value.y) {
    score.value++
    food.value = {
      x: Math.floor(Math.random() * gridSize),
      y: Math.floor(Math.random() * gridSize)
    };
  } else {
    snake.value.pop(); // Supprimer la queue
  }
};

const resetGame = () => {
  snake.value = [{ x: 5, y: 5 }] ;
  direction.value = 'right'
  score.value = 0
}
// Changer la direction avec les touches
const changeDirection = (event) => {
  if (event.key === 'ArrowUp' && direction.value !== 'down') direction.value = 'up';
  if (event.key === 'ArrowDown' && direction.value !== 'up') direction.value = 'down';
  if (event.key === 'ArrowLeft' && direction.value !== 'right') direction.value = 'left';
  if (event.key === 'ArrowRight' && direction.value !== 'left') direction.value = 'right';
  if (event.key === '') togglePause ()
};

const togglePause = () => {
  isPaused.value = !isPaused.value
}
onMounted(() => {
  window.addEventListener('keydown', changeDirection);
  gameInterval =setInterval(moveSnake, 300);
//  (moveSnake, 300);
});

onUnmounted(() => {
  window.removeEventListener('keydown', changeDirection);
  clearInterval(gameInterval)
});
</script>

<style>
.game-container {
  display: grid;
  grid-template-rows: repeat(10, 30px);
  gap: 2px;
  justify-content: center;
  margin-top: 20px;
}
h2 {
  text-align: left;
  font-size: 2em;
  margin-top: -20px;
}
.row {
  display: grid;
  grid-template-columns: repeat(10, 30px);
  gap: 2px;
}

.cell {
  width: 30px;
  height: 30px;
  background-color: #ddd;
  border-radius: 5px;
}

.snake {
  background-color: green;
}

.food {
  background-color: red;
}
</style>
