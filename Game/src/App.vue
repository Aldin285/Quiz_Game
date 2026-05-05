<script setup>
  import Footer from './Views/Footer/Footer.vue';
import GameDisplay from './Views/GameDisplay/GameDisplay.vue';
import Button from './Components/Button/Button.vue';
  import data from "./data/quiz.json"

  import { ref, computed, reactive } from 'vue';

  // Affichage
  const getStarted = ref(true);
  const displayElements = ref(false);
  const displayEndScreen = ref(false);

  // Fonction pour résoudre le chemin de l'image
  const baseUrl = import.meta.env.BASE_URL;
  const resolveImagePath = (path = '') => {
    if (!path) return '';
    if (/^https?:\/\//i.test(path)) return path;
    return `${baseUrl}${path.replace(/^\/+/, '')}`;
  };

  // Organisation des questions
  // Générer un nombre aléatoire
   function getRandomInt(max) {
    return Math.floor(Math.random() * max);
  }
  // Liste des questions et la limite de questions à afficher
  let selectedQuestions = reactive([]);
  const questionLimits = 1;

  // Sélection des questions
  function selectQuestions() {
    selectedQuestions.length = 0; // Clear array while maintaining reactivity
    while(selectedQuestions.length < questionLimits){
      const randInt = getRandomInt(data.length);
      if(!selectedQuestions.includes(randInt)){
        selectedQuestions.push(randInt);
      }
    }
    //  console.log("Selected Questions : ", selectedQuestions);
  }
  selectQuestions();
 
  // La position de la question actuelle dans la liste des questions sélectionnées
  const currentQuestionPosition = ref(questionLimits - 1);

  // Id de la bonne réponse pour la question actuelle
  const currentQuestionIndex = computed(() => selectedQuestions[currentQuestionPosition.value]);
  const answerId = ref(data[currentQuestionIndex.value].answerId);

  // Score
  const totalScore = ref(0);

  // Fonction pour passer à la question suivante et afficher le score 
  function NextQuestion() {
    if(currentQuestionPosition.value > 0){
      currentQuestionPosition.value--;
      answerId.value = data[selectedQuestions[currentQuestionPosition.value]].answerId;
    }else{
      displayElements.value = false;
      displayEndScreen.value = true;
      console.log("Quiz finished")
    }
  }

  // Reset game
  function ResetGame(){
    getStarted.value = true;
    displayEndScreen.value = false;

    selectQuestions();

    currentQuestionPosition.value = questionLimits - 1;
    answerId.value = data[selectedQuestions[currentQuestionPosition.value]].answerId;

    totalScore.value = 0;
  }

  function handleNextQuestion() {
    NextQuestion();
  }

 
 
</script>

<template>
  <div class="screen-root">

    <!-- Ecran de jeu -->
    <transition name="fade" mode="out-in">
      <div v-if="displayElements" key="display">
        <GameDisplay 
          :currentQuestion="data[currentQuestionIndex]"
          :answerId="answerId"
          :imagePath="resolveImagePath"
          @nextQuestion="handleNextQuestion"
          @scoreUpdate="(score)=>totalScore = score"
        />
         
      </div>
    </transition>

    <!-- Ecran démarrage -->
    <transition name="fade" mode="out-in">
      <div v-if="getStarted" class="getStarted" key="start">
        <h1>Welcome to the Quiz Game!</h1>
        <Button
          text="Get Started"
          @click="{ displayElements = true; getStarted = false }"
        />
      </div>
    </transition>

    <!-- Ecran fin de la partie -->
    <transition name="fade" mode="out-in">
      <div v-if="displayEndScreen" class="getStarted" key="end">
        <h1>Your final score is :</h1>
        <h1> {{ totalScore }}</h1>
        <!-- <button @click="ResetGame"  class="button-82-pushable">
        <span class="button-82-shadow"></span>
        <span class="button-82-edge"></span>
        <span class="button-82-front text">
          Play again
        </span>
        </button> -->
        <Button
          text="Play again"
          @click="ResetGame"
        />
      </div>
    </transition>

     <!-- Footer -->
    <Footer />

  </div>
</template>

<style scoped>
 
@import url("./global.css");

</style>
