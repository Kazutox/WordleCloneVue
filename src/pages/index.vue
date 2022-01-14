<script setup lang="ts">
import { Ref } from 'vue';

declare type GameCell = {
  color: string,
  char: string
}

const secretWord: string = "tangy";
const inputWord: Ref<string> = ref('');

const gameGridData: Ref<GameCell[][]> = ref([[]]);
const gameCount: Ref<number> = ref(0);

//Loaded words from wordlist.txt
let words: string[] = [];

//Load the wordlist text file onMount (lifecycle hook)
onMounted(()=>{
  fetch(`/wordlist.txt`).then(async result =>{
    const data = (await result.text()).split('\r\n');
    const fiveChar = data.filter(string =>{
      const length = string.length;
      if(length == 5){
        return true;
      }
      return false;
    });

    words = fiveChar;
  });
});

function handleButtonClick(){
  //1. Check for valid word, if not valid stop early.
  const validWord = isValidWord(inputWord.value)
  if(!validWord){
    window.alert("Word is invalid, please try again!")
    inputWord.value = "";
    return;
  }

  //2.Validate the guess and populate gridData for the round
  const data = gameGridData.value;
  data[gameCount.value] = validateWordGuess();
  gameGridData.value = data;

  //3. Increase counter and reset input value
  gameCount.value += 1;
  inputWord.value = "";

  //4. You only have 6 tries, after 6 tries the game will restart (same word)
  if(gameCount.value === 6){
    window.alert("Game over, please try again");
    gameGridData.value = [];
    gameCount.value = 0;
  }
}

function validateWordGuess(): GameCell[]{
  let result = [];
  for(let i = 0; i < 5; i++){
    const guessChar = inputWord.value.charAt(i).toLowerCase();
    const secretChar = secretWord.charAt(i).toLowerCase();
    if(guessChar === secretChar){
      result.push({color: 'green', char: guessChar});
    }else if(secretWord.includes(guessChar)) { 
      result.push({color: 'yellow', char: guessChar});
    }else{
      result.push({color: 'gray', char: guessChar});
    }
  }
  return result;
}

function isValidWord(word: string){
  return words.includes(word.toLowerCase());
}
</script>

<template>
<div class="game-container">
  <div class="menu-container">
    <h1 class="game-title">Wordle Clone</h1>
    <h2 class="game-count">GameCount: {{gameCount}}</h2>
    <input placeholder="Enter your guess" v-model="inputWord" class="guess-input" maxlength="5"/>
    <button v-on:click="handleButtonClick()" class="guess-submit">Submit</button>
  </div>
  <div class="game-window">
    <div v-for="row of gameGridData">
      <div
        v-for="cell of row"
        :class="`game-cell bg-${cell.color}-500`"
      >
        {{cell.char}}
      </div>
    </div>
  </div>
</div>
</template>

<style scoped>
.game-container{
  @apply container mr-auto ml-auto w-96 p-2;
}
.menu-container{
  @apply bg-white text-black p-2 w-96 grid grid-cols-1 gap-2;
}
.game-title{
  @apply text-2xl mb-2 row-start-1;
}
.game-count{
  @apply row-start-2;
}
.guess-input{
  @apply bg-gray-300 row-start-3 p-1 rounded-md;
}

.guess-submit{
  @apply bg-green-500 p-1 rounded-md;
}
.game-window{
  @apply p-2 w-96 text-black bg-white grid grid-rows-6;
  
}

.game-window div{
  @apply grid;
  grid-template-columns: repeat(5,72.8px);
}

.game-cell{
  @apply h-6 border border-collapse;
}
</style>