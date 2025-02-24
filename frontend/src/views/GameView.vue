<script setup lang="ts">
import GameBoard from '../components/GameBoard.vue'
import { socket, state } from '../socket';
import { computed } from 'vue';
import { useRouter } from 'vue-router';
import { watch } from 'vue';

const router = useRouter();
const gameStarted = computed(() => state.gameStarted);
const gameOver = computed(() => state.gameOver);

socket.emit('is in game', (res) => {
  console.log('is in game', res);
  if (!res.inGame) router.push('/');
});

watch(() => state.opponentDisconnected, (disconnected) => {
  if (disconnected) router.push('/');
  state.opponentDisconnected = false;
});
</script>

<template>
  <main>
    <div class="boards">
      <GameBoard :my-board="true" />
      <GameBoard :opponent-board="true" />
    </div>
    <div class="text">
      <div v-if="gameOver">
        Game Over!
      </div>
      <div v-else>
        <p v-if="gameStarted">Attack your opponents ships!</p>
        <p v-else>Place ships on your battleground!</p>
      </div>
    </div>
  </main>
</template>

<style scoped>
.boards {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 20px;
}

.text {
  margin-top: 50px;
  text-align: center;
}
</style>
