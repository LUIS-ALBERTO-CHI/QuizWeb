<template>
  <div class="quiz-container" v-if="started">
    <div class="card" v-if="currentIndex < questions.length">
      <div class="timer-bar-container">
        <div class="timer-bar" :style="{ width: `${timeLeft}%` }"></div>
        <span class="timer-text">{{ timeLeft }} segundos</span>
      </div>
      <div class="question">
        <h2>{{ currentQuestion.questionText }}</h2>
        <p>Pregunta {{ currentIndex + 1 }} de {{ questions.length }}</p>
      </div>
      <div class="options">
        <button
          v-for="option in currentOptions"
          :key="option.optionId"
          :class="{
            'correct': selectedOption !== null && option.isCorrect,
            'incorrect': selectedOption === option.optionId && !option.isCorrect
          }"
          @click="selectOption(option.optionId)"
          :disabled="selectedOption !== null"
        >
          {{ option.optionText }}
          <span v-if="selectedOption !== null && option.optionId === selectedOption">
            {{ option.isCorrect ? '✔️' : '❌' }}
          </span>
        </button>
      </div>
      <div v-if="selectedOption !== null">
        <button @click="nextQuestion">Siguiente</button>
      </div>
    </div>
    <div class="overlay" v-else>
      <h2>Quiz terminado!</h2>
      <p>Aciertos: {{ correctAnswers }} / {{ questions.length }}</p>
      <p>{{ getCompletionMessage() }}</p>
      <button @click="restartQuiz">Reiniciar Quiz</button>
    </div>
  </div>
  <div v-else class="start-overlay">
    <button @click="startQuiz">Iniciar Quiz</button>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      questions: [],
      currentIndex: 0,
      selectedOption: null,
      timeLeft: 100,
      intervalId: null,
      correctAnswers: 0,
      started: false,
    };
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentIndex].question;
    },
    currentOptions() {
      return this.questions[this.currentIndex].options;
    }
  },
  methods: {
    async fetchQuestions() {
      try {
        const response = await axios.get('http://localhost:5000/api/questions/getQuestions');
        const allQuestions = response.data;
        this.questions = this.processQuestions(allQuestions).slice(0, 10);
        this.startTimer();
      } catch (error) {
        console.error("Error fetching questions:", error);
      }
    },
    processQuestions(questions) {
      const map = new Map();
      questions.forEach(q => {
        if (!map.has(q.questionId)) {
          map.set(q.questionId, {
            question: {
              questionId: q.questionId,
              questionText: q.questionText
            },
            options: []
          });
        }
        map.get(q.questionId).options.push({
          optionId: q.optionId,
          optionText: q.optionText,
          isCorrect: q.isCorrect
        });
      });
      return Array.from(map.values());
    },
    async selectOption(optionId) {
      this.selectedOption = optionId;
      const response = await axios.post('http://localhost:5000/api/questions/submitAnswer', {
        questionId: this.currentQuestion.questionId,
        optionId
      });
      const isCorrect = response.data.correct;
      if (isCorrect) {
        this.correctAnswers++;
      }
      clearInterval(this.intervalId);
    },
    nextQuestion() {
      this.selectedOption = null;
      this.timeLeft = 100;
      this.currentIndex++;
      if (this.currentIndex < this.questions.length) {
        this.startTimer();
      }
    },
    startTimer() {
      this.intervalId = setInterval(() => {
        if (this.timeLeft > 0) {
          this.timeLeft -= 1;
        } else {
          clearInterval(this.intervalId);
          this.nextQuestion();
        }
      }, 100);
    },
    startQuiz() {
      this.started = true;
      this.fetchQuestions();
    },
    restartQuiz() {
      this.started = false;
      this.currentIndex = 0;
      this.correctAnswers = 0;
      this.selectedOption = null;
      this.timeLeft = 100;
      clearInterval(this.intervalId);
    },
    getCompletionMessage() {
      if (this.correctAnswers === 0) {
        return "Lo siento, conseguiste 0.";
      } else if (this.correctAnswers <= 4) {
        return "Te falta estudiar.";
      } else if (this.correctAnswers <= 6) {
        return `Qué bien, sacaste ${this.correctAnswers}.`;
      } else if (this.correctAnswers <= 8) {
        return `¡Felicitaciones, sacaste ${this.correctAnswers}!`;
      } else {
        return `¡Excelente, sacaste ${this.correctAnswers}!`;
      }
    }
  },
  created() {
   
  }
};
</script>

<style scoped>
.quiz-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background: #f2f2f7;
}
.card {
  background: #ffffff;
  padding: 20px;
  border-radius: 20px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  text-align: center;
  width: 80%;
  max-width: 600px;
}
.timer-bar-container {
  position: relative;
  width: 100%;
  height: 10px;
  background: #d1d1d6;
  border-radius: 5px;
  margin-bottom: 20px;
}
.timer-bar {
  height: 100%;
  background: #34c759;
  border-radius: 5px;
  transition: width 0.1s linear;
}
.timer-text {
  position: absolute;
  right: 10px;
  top: -25px;
  font-size: 14px;
  color: #1c1c1e;
}
.question h2 {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  font-size: 24px;
  color: #1c1c1e;
  margin-bottom: 10px;
}
.question p {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  font-size: 18px;
  color: #1c1c1e;
  margin-bottom: 20px;
}
.options button {
  display: block;
  width: 100%;
  margin: 10px 0;
  padding: 15px;
  background: #f2f2f7;
  border: 1px solid #d1d1d6;
  border-radius: 10px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  font-size: 18px;
  color: #1c1c1e;
  cursor: pointer;
  transition: background 0.3s, border 0.3s;
}
.options button.correct {
  background: #34c759;
  border-color: #34c759;
  color: white;
}
.options button.incorrect {
  background: #ff3b30;
  border-color: #ff3b30;
  color: white;
}
.options button:disabled {
  cursor: not-allowed;
}
.start-overlay, .overlay {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  flex-direction: column;
  background: #f2f2f7;
}
.start-overlay button, .overlay button {
  padding: 15px 30px;
  font-size: 18px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 10px;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  cursor: pointer;
  transition: background 0.3s;
}
.start-overlay button:hover, .overlay button:hover {
  background: #0056b3;
}
.overlay h2 {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  font-size: 24px;
  color: #1c1c1e;
  margin-bottom: 20px;
}
.overlay p {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  font-size: 18px;
  color: #1c1c1e;
  margin-bottom: 20px;
}
</style>
