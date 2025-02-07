<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <div class="message">
      
      <p>{{ empire }}</p>
    </div>

    <button type="button" @click="submitPlayerScore('player1score')">Player1 - {{ player1score }}</button>
    <button type="button" @click="submitPlayerScore('player2score')">Player2 - {{ player2score}}</button>
    
  </div>
</template>

<script setup lang="ts">

  import { ref, onMounted } from 'vue'
  import axios from 'axios'

  defineProps<{ msg: string }>()
  let player1score = ref(0);
  let player2score = ref(0);
  let empire = ref('');

  const scores = ref([]);

  onMounted(() => {
    const savedScores = JSON.parse(localStorage.getItem("scores"));
    if (savedScores) {
      scores.value = savedScores;
    }
  });
 
  const submitPlayerScore = async (playerWhoScored) => {
    if(playerWhoScored === 'player1score') {
      player1score.value++;
    } else {
      player2score.value++;
    }

    //api call to update scores
    const response = await axios.post('http://localhost:8000/backend/', {
      message: playerWhoScored,
      player1score: player1score.value,
      player2score: player2score.value
    });

    //display the empire results
    empire.value = response.data.message;

    //reset scores and localStorage round Completed
    if(response.data.complete) {
      player1score.value = 0,
      player2score.value = 0,
      localStorage.clear();
      scores.value = null;
      return;
    }
    
    //add round score to localStorage
    scores?.value?.push({
      id: generateUniqueId(),
      score: response.data.message,
      player1: player1score.value,
      player2: player2score.value
    });

    saveScoresToLocalStorage();
  }

  const generateUniqueId = () => {
    return Math.floor(Math.random() * 100)
  }

  const saveScoresToLocalStorage = () => {
    localStorage.setItem('scores', JSON.stringify(scores.value));
  }

</script>




