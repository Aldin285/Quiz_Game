<script setup>
  import OptionBox from '../../Components/OptionBox/OptionBox.vue';
  import question from '../../Components/Question/Question.vue';
  import timer from '../../Components/Timer/Timer.vue';
  import score from '../../Components/Score/Score.vue';

  import { ref, onMounted, watch } from 'vue';

  // Props from parent (App.vue)
  const props = defineProps({
    currentQuestion: {
      type: Object,
      required: true
    },
    answerId: {
      type: Number,
      required: true
    },
    imagePath: {
      type: Function,
      required: true
    }
  });

  // Emit events to parent
  const emit = defineEmits(['checkAnswer', 'nextQuestion', 'timeOut', 'scoreUpdate']);

  // Pour la réponse sélectionnée
  const selectedAnswer = ref(null);

  // Pour les réponses
  const rightAnswerStyle = ref({});
  const wrongAnswerStyle = ref({});
  const disableButtons = ref(false);

  // Chrono
  const timeLeft = ref();
  const stopTimer = ref(false);
  const restartTimer = ref(false);
  const dashLength = ref(0);
  const totalDashLength = ref(0);

  // Score
  const checkAnswer = ref(false);
  const totalScore = ref(0);

  // Fonction pour passer à la question suivante
  function NextQuestion() {
    setTimeout(() => {
      emit('nextQuestion');
      
      // Réinitialise les styles et les variables
      rightAnswerStyle.value = {};
      wrongAnswerStyle.value = {};
      disableButtons.value = false;
      selectedAnswer.value = null;
      stopTimer.value = false;
      checkAnswer.value = false;
      restartTimer.value = true;
      setTimeout(() => {
        restartTimer.value = false;
      }, 100);
    }, 3000);
  }

  function CheckAnswer(e) {
    // Récupère l'id de la réponse sélectionnée
    selectedAnswer.value = e.target.id;

    // Changement de style par rapport à la réponse sélectionnée
    // Bonne réponse
    if(props.answerId == selectedAnswer.value){
      rightAnswerStyle.value = {
        border: '2px solid green',
        backgroundColor: 'lightgreen',
        cursor: 'not-allowed',
      };

      wrongAnswerStyle.value = {
        cursor: 'not-allowed',
      };

      checkAnswer.value = true;
    }else{
      // Mauvaise réponse
      rightAnswerStyle.value = {
        border: '2px solid green',
        backgroundColor: 'lightgreen',
        cursor: 'not-allowed',
      };
      
      wrongAnswerStyle.value = {
        border: '2px solid red',
        backgroundColor: '#ff7f7f',
        cursor: 'not-allowed',
      };
    }

    // Désactive les boutons après une sélection
    disableButtons.value = true;
    // Arrête le chrono après une sélection
    stopTimer.value = true;

    // Emit check answer event
    emit('checkAnswer', {
      isCorrect: props.answerId == selectedAnswer.value
    });

    // Changer la question
    NextQuestion();
  }

  // Annule la séléction de réponse quand le temps est écoulé
  function TimeOut() {
      // Affiche la bonne réponse 
      rightAnswerStyle.value = {
        border: '2px solid green',
        backgroundColor: 'lightgreen',
        cursor: 'not-allowed',
      };
      
      // Affiche les mauvaises réponses
      wrongAnswerStyle.value = {
        border: '2px solid red',
        backgroundColor: '#ff7f7f',
        cursor: 'not-allowed',
      };

      // Désactive les boutons 
      disableButtons.value = true;

      emit('timeOut');
      
      // Changer la question
      NextQuestion();
    }

  watch(timeLeft, (newValue) => {
    if (newValue === 0 && !stopTimer.value) {
      TimeOut();
    }
  });
</script>

<template>
  <div class="displayElements">

    <!-- Timer -->
    <timer :timeLimit="50"
      :stop="stopTimer" 
      :restart="restartTimer"
      @timeLeft='(tl) => timeLeft = tl'
      @elapsedDash="(dash, length)=>{
        dashLength = dash;
        totalDashLength = length;
      }" 
    />

    <!-- Score -->
    <score 
      :dashLeft="dashLength" 
      :totalDash="totalDashLength" 
      :check="checkAnswer" 
      @finalScore="(score) =>{emit('scoreUpdate', score)}" 
    />

    <!-- Question -->
    <question 
      :question="currentQuestion.question"
      :imgURL="imagePath(currentQuestion.imageURL)" 
    />
    
    <!-- Selection des réponses -->
    <div class="answerSelect">
      <div v-for="option in currentQuestion.answers" :key="option.id">
        <OptionBox 
          :id="option.id"
          :answer="option.answerText"
          :style="answerId === option.id ? rightAnswerStyle : wrongAnswerStyle"
          @click="CheckAnswer" 
          :disabled="disableButtons"
        />
      </div>
    </div>

   
  </div>
</template>

<style scoped>
  @import url("./GameDisplay.css");
</style>
