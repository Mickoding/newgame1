<template>
    <v-container class="mt-10">
    <v-card class="mx-auto" max-width="400">
      <v-container class="d-flex justify-center align-center">
        <v-icon size="100">mdi-calculator</v-icon>
      </v-container>
      <v-card-title class="text-center">Calculator</v-card-title>
    <v-card-text>
      <v-row class="mb-3">
        <v-col cols="12">
          <v-text-field v-model="display" outlined readonly class="text-right"></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col v-for="button in buttons" :key="button" cols="3">
          <v-btn block @click="handleButtonClick(button)">{{ button }}</v-btn>
        </v-col>
      </v-row>
      <v-divider class="my-4"></v-divider>
      <v-list dense>
        <v-list-item v-for="(calculation, currentCalc) in history" :key="currentCalc">
          <v-list-item-content>
            <v-list-item-title>{{ calculation }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
      <v-btn block color="pink-lighten-1" @click="clearCalcHistory">Clear History</v-btn>
    </v-card-text>
    </v-card>
  </v-container>
</template>


<script setup>
import { ref} from 'vue';

const display = ref('');
const history = ref([]);
let lastOperation='';

const buttons = [
  'x^2', '%', '√', '+',
  '7', '8', '9', '-',
  '4', '5', '6', '*',
  '1', '2', '3', '/',
  'AC', '0', '=' 
];

function handleButtonClick(button) {
  if (button === 'AC') {
    display.value = '';
  } else if (button === '=') {
  try {
    const result = eval(display.value);
    const operation = display.value + ' = ' + result;
    display.value = operation;
    addToHistory(operation);
    setTimeout(() => {
      display.value = ''; // E clear ang display
    }, 500); // Delay time para ma automatically clear
  } catch (error) {
    display.value = 'Error';
  }
} else if (button === 'x^2') {
    display.value = Math.pow(Number(display.value), 2).toString();
  } else if (button === '%') {
    display.value = (Number(display.value) / 100).toString();
  } else if (button === '√') {
    display.value = Math.sqrt(Number(display.value)).toString();
  } else {
    display.value += button;
    lastOperation = button;
  }
}

function addToHistory(calculation) {
  if (history.value.length >= 10) {  
    history.value.shift(); 
  }
  history.value.push(calculation); 
}

function clearCalcHistory() {
  history.value = [];
}
</script>

<style scoped> 
.text-right {
  text-align: right;
}
</style>