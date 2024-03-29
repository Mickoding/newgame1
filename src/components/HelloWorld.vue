<template>
  <div class="container">
    <h2 class="player">{{ handleMessage }}</h2>
    <v-container v-if="!gameStarted">
      <v-row>
        <v-col>
          <v-text-field v-model="playerOneName" label="Player One Name" outlined></v-text-field>
        </v-col>
        <v-col>
          <v-text-field v-model="playerTwoName" label="Player Two Name" outlined></v-text-field>
        </v-col>
      </v-row>
      <v-btn color="blue" @click="startGame" :disabled="playerOneName === '' || playerTwoName === ''">Start Game</v-btn>
    </v-container>
    <v-container v-if="gameStarted && !selectedGameMode">
      <h3>Select Game Mode:</h3>
      <v-btn color="primary" @click="selectGameMode('Random')">Random Game Mode</v-btn>
    </v-container>
    <v-btn color="grey" @click="handleReset" class="reset-btn">Reset?</v-btn>
    <v-btn color="red" @click="clearTicHistory" class="reset-btn">Clear History</v-btn>

    <v-container v-if="selectedGameMode === 'Random'">
      <v-row>
        <v-col v-for="row in 5" :key="row">
          <v-row>
            <v-col v-for="column in 5" :key="column">
              <v-btn dark class="tiles" @click="selectTile(row, column)">
                {{ tileValue(row, column) !== '' ? tileValue(row, column) : '[]' }}
              </v-btn>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
    </v-container>

    <h3>Score History:</h3>
    <v-list dense>
      <v-list-item v-for="(score, index) in scores" :key="index">
        <v-list-item-content>
          <v-list-item-title>{{ score.player }}: {{ score.score }}</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
    </v-list>

    <h3>Highest Scorer:</h3>
    <p>{{ highestScorer }}</p>
  <v-divider></v-divider>
  <v-text-field v-model="message" label="Message" outlined></v-text-field>
    <v-btn color="primary" @click="sendMessage">Send</v-btn>
  <v-card class="chat-box">
      <v-list>
        <v-list-item v-for="(msg, index) in messages" :key="index">
          <v-list-item-content>
            <v-list-item-title>{{ msg }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-card>
    <v-btn color="red" @click="clearChat">Clear Chat</v-btn>
  </div>
<v-divider></v-divider>
  <v-btn @click="toggleCalculator">Use Calculator</v-btn>
<v-divider></v-divider>
  <v-btn v-if="showCalculator" @click="hideCalculator">Hide Calculator</v-btn>
    <Calculator v-if="showCalculator" />
  <v-btn @click="toggleEmojiWheel">Toggle Emoji Wheel</v-btn>
    <EmojiWheel v-if="showEmojiWheel" @emoji-selected="handleEmojiSelected" />
    <v-btn color="orange" @click="undoMove" :disabled="undoStack.length === 0">Undo Move</v-btn>

</template>

<script setup>
import { ref, computed } from 'vue';
import Calculator from './Calculator.vue';
import EmojiWheel from './EmojiWheel.vue';

const playerOneName = ref('');
const playerTwoName = ref('');
const gameStarted = ref(false);
const selectedGameMode = ref('');
const showSymbolMode = ref(false);
const currentPlayer = ref(playerOneName.value);
const gameOver = ref('');
const scores = ref([]);
const highestScorer = ref('No highest scorer yet');
const usedNumbers = ref([]);
const tics = ref([]);
const showCalculator = ref(false);
const showEmojiWheel = ref(false);
const message = ref('');
const messages = ref([]);
const undoStack = ref([]);

const selectGameMode = (mode) => {
  selectedGameMode.value = mode;
  if (mode === 'Random') {
    tics.value = randomTileValues();
  }
}

const randomTileValues = () => {
  const numbers = Array.from({ length: 25 }, (_, index) => index < 5 ? Math.floor(Math.random() * 25) + 1 : '');
  for (let i = numbers.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [numbers[i], numbers[j]] = [numbers[j], numbers[i]];
  }
  return numbers.map((value, index) => ({ value, clicked: false }));
};

const tileIndex = (row, column) => (row - 1) * 5 + column - 1;

const tileValue = (row, column) => {
  const index = tileIndex(row, column);
  return tics.value[index]?.value || '';
}

const handleReset = () => {
  resetGame();
  showSymbolMode.value = false;
}

const undoMove = () => {
  if (undoStack.value.length > 0) {
    const lastMove = undoStack.value.pop();
    const { row, column, previousValue } = lastMove;
    tics.value[tileIndex(row, column)].value = previousValue;
    redoStack.value.push(lastMove);
  }
}

const recordMove = (row, column, previousValue) => {
  undoStack.value.push({ row, column, previousValue });
}

const selectTile = (row, column) => {
  if (gameOver.value) return;
  const index = tileIndex(row, column);
  if (tics.value[index].value !== '') {
    window.alert('This tile already has a number in it. Please choose another tile.');
    return;
  }
  let number = prompt(`Player ${currentPlayer.value}, enter a number between 1 and 25:`);
  if (!number || isNaN(number) || number.includes('0')) {
    window.alert('You cannot input a number containing 0. Please input another number.');
    return;
  }
  let parsedNumber = parseInt(number);
  if (parsedNumber < 1 || parsedNumber > 25) {
    window.alert('Number must be between 1 and 25. Please choose a different number.');
    return;
  }
  recordMove(row, column, tics.value[index].value);
  tics.value[index].value = parsedNumber;
  tics.value[index].clicked = true;
  usedNumbers.value.push(parsedNumber);
  currentPlayer.value = currentPlayer.value === playerOneName.value ? playerTwoName.value : playerOneName.value;
  checkWinningCondition(); 
}

const recordScore = (playerName) => {
  const scoreIndex = scores.value.findIndex(score => score.player === playerName);
  if (scoreIndex !== -1) {
    scores.value[scoreIndex].score++;
  } else {
    scores.value.push({ player: playerName, score: 1 });
  }
  updateHighestScorer();
}

const checkWinningCondition = () => {
  const winningSets = [
    [0, 1, 2, 3, 4], [5, 6, 7, 8, 9], [10, 11, 12, 13, 14], [15, 16, 17, 18, 19], [20, 21, 22, 23, 24], // Horizontal sets
    [0, 5, 10, 15, 20], [1, 6, 11, 16, 21], [2, 7, 12, 17, 22], [3, 8, 13, 18, 23], [4, 9, 14, 19, 24], // Vertical sets
    [0, 6, 12, 18, 24], [4, 8, 12, 16, 20] // Diagonal sets
  ];

  for (const set of winningSets) {
    const tileValues = set.map(index => tics.value[index].value);
    const sum = tileValues.reduce((acc, curr) => acc + curr, 0);
    if (sum === 55 && tileValues.every(value => value !== '')) {
      const winningPlayer = currentPlayer.value === playerOneName.value ? playerTwoName.value : playerOneName.value;
      const winningTiles = set.map(index => tics.value[index].value).join(', ');
      window.alert(`Player ${winningPlayer} wins with tiles ${winningTiles}`);
      recordScore(winningPlayer);
      resetGame();
      return;
    }
  }

  const allTilesTaken = tics.value.every(tile => tile.value !== '');
  if (allTilesTaken) {
    window.alert('No Winner');
    resetGame();
  }
}

const resetGame = () => {
  currentPlayer.value = playerOneName.value;
  gameOver.value = '';
  gameStarted.value = false;
  usedNumbers.value = [];
  tics.value = [];
  selectedGameMode.value = '';
}

const updateHighestScorer = () => {
  const highest = scores.value.reduce((prev, current) => (prev.score > current.score) ? prev : current);
  highestScorer.value = `${highest.player}: ${highest.score}`;
}

const clearTicHistory = () => {
  scores.value = [];
  highestScorer.value = 'No highest scorer yet';
}

const startGame = () => {
  gameStarted.value = true;
  currentPlayer.value = playerOneName.value;
}
const handleMessage = computed(() => {
  if (gameOver.value === 'No Winner') {
    return "All tiles are taken, but no winner!";
  } else if (gameOver.value === 'One' || gameOver.value === 'Two') {
    return `Player ${playerOneName.value} wins!`;
  } else {
    return `It's ${currentPlayer.value}'s turn.`;
}});


const toggleCalculator = () => {
  showCalculator.value = !showCalculator.value;
}


const hideCalculator = () => {
  showCalculator.value = false;
}

const toggleEmojiWheel = () => {
  showEmojiWheel.value = !showEmojiWheel.value;
}

const handleEmojiSelected = (emoji) => {
  message.value += emoji;
}

const sendMessage = () => {
  if (message.value.trim() !== '') {
    messages.value.push(message.value);
    message.value = '';
  }
}

const clearChat = () => {
  messages.value = [];
}

</script>
<style scoped>
.container {
  text-align: center;
  background-color: aqua;
  max-width: 800px;
  margin: 0 auto;
}

.player {
  margin-bottom: 20px;
}

.reset-btn {
  margin-bottom: 20px;
}

.tiles {
  width: 40px;
  height: 40px;
  padding: 0;
  margin: 5px;
}

.dark-text {
  color: #212121;
}
</style>