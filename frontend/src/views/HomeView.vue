<script setup lang="ts">
import { state, socket } from '../socket';
import { ref, watch } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

const lookingForGame = ref(false);

function startGame() {
  lookingForGame.value = true;
  socket.emit('join game');
}

watch(() => state.foundGame, (foundGame) => {
  if (foundGame) router.push('/game');
  state.foundGame = false;
});

</script>

<template>
  <div class="content">
    <h1 class="title">BATTLESHIP</h1>
    <button @click="startGame">Play</button>
    <p v-if="lookingForGame">Looking for game...</p>
  </div>
</template>

<style scoped>
.content {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  height: 80vh;
}

.title {
  margin-bottom: 10px;
  letter-spacing: 5px;
  font: monospace;
}

button {
  margin-bottom: 15px;
}
</style>
