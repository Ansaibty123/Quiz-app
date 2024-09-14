<template>
  <div class="container vh-100 p-3 text-white d-flex flex-column ">
    <h1 class="h1 p-2 p-md-5 d-flex justify-content-center ">Quiz App</h1>

    <!-- Quiz Content -->
    <div v-if="!showResults">
      <div v-if="questions && questions.length > 0" >
        <div class="d-flex justify-content-between pb-3 flex-row gap-2 border-bottom border-light ">
          <div class="d-flex flex-row gap-2">
            <span class="text-warning text-center custom-font">Question</span> <span class=" custom-font"> {{ currentQuestionIndex + 1 }} / {{ questions.length }}</span>
          </div>
          <div class="d-flex gap-2 w-50 p-0">
            <input v-model.number="questionJump" type="number" placeholder="Q. no" class="w-50 p-0 p-md-3">
            <button @click="jumpQuestionNum" class="btn btn-primary w-50">Submit</button>
          </div>
        </div>
        <div v-if="currentQuestion">
          <div class="pb-4">
            <span class="text-warning fs-2">{{ currentQuestionIndex + 1 }} .</span>
            <span class="fs-2">{{ currentQuestion.Question }}</span>
          </div>
          <div class="d-flex gap-3 flex-column pb-4">
            <div v-for="(option, index) in currentQuestion.Options" :key="index">
              <button class="btn btn-light fs-4" v-on:click="checkAnswer(option)" :style="buttonStyle(option)">
                {{ option }}
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Exam and Explanation -->
      <div v-if="currentQuestion && currentQuestion.Exam" class="pt-2 pb-2 pe-2 text-end">
        <span class="text-bg-info  pb-2 pt-2  ">Exam => {{ currentQuestion.Exam }}</span>
      </div>
      <div v-if="currentQuestion && currentQuestion.Explanation" class=" pt-2 pb-2   ">
        <span class="text-bg-info pb-2 pt-2">Explanation => {{ currentQuestion.Explanation }} </span>
      </div>

      <!-- Navigation Buttons -->
      <div v-if="questions" class="d-flex gap-4 pt-3 pb-3">
        <button class=" btn btn-dark " @click="previousPage" :disabled="currentQuestionIndex === 0">Previous</button>
        <button class=" btn btn-success" @click="nextPage"
          :disabled="currentQuestionIndex > questions.length - 1">Next</button>
      </div>
    </div>

    <!-- Results -->
    <div v-if="showResults" class="d-flex flex-column justify-content-center align-items-center vh-100">
      <div class="text-center text-white">
        <div class="display-1">Quiz Results</div>
        <div class="fs-1">Score: {{ score }} / {{ questions.length }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import csv from 'csvtojson';

export default {
  data() {
    return {
      questions: null,
      currentQuestionIndex: 0,
      score: 0,
      showResults: false,
      selectedOption: null,
      questionJump: null,
    };
  },
  methods: {
    async fetchAndConvert() {
      const csvUrl = "https://gist.githubusercontent.com/VishalBty/780fa362a5be0770ebdfa05f584ed039/raw";
      try {
        const response = await axios.get(csvUrl);

        const jsonObj = await csv().fromString(response.data)

        this.questions = jsonObj.map(({ Question, A, B, C, D, Answer, Exam, Explanation }) => ({
          Question: this.removeNumbers(Question),
          Exam,
          Explanation,
          Options: [A, B, C, D],
          CorrectAnswer: Answer
        }));
        this.loadQuiz()

        console.log('CSV to JSON conversion successful:', jsonObj);
      } catch (error) {
        console.log(error)
      }
    },
    checkAnswer(option) {
      this.selectedOption = option;
      if (option === this.currentQuestion.CorrectAnswer) {
        this.score++
      }
    },
    buttonStyle(option) {
      if (this.selectedOption === option) {
        return option === this.currentQuestion.CorrectAnswer ?
          "background-color : green ; color : white" :
          "background-color : red ; color : white"
      }
      return ''
    },
    resetOption() {
      this.selectedOption = null;
    },
    nextPage() {
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++
        this.resetOption();
        this.saveQuiz()
      } else {
        this.showResults = true;
      }
    },
    previousPage() {
      if (this.currentQuestionIndex > 0) {
        this.currentQuestionIndex--
        this.resetOption();
      }
    },
    removeNumbers(question) {
      return question.replace(/^\d+\.\s*/, '');
    },
    saveQuiz() {
      const quiz = {
        currentQuestionIndex: this.currentQuestionIndex,
        score: this.score,
      };
      localStorage.setItem("quizIndex", JSON.stringify(quiz));
    },
    loadQuiz() {
      const savedQuiz = localStorage.getItem("quizIndex");
      if (savedQuiz) {
        const { currentQuestionIndex, score } = JSON.parse(savedQuiz);
        this.currentQuestionIndex = currentQuestionIndex;
        this.score = score;
      }
    },
    jumpQuestionNum() {
      if (this.questionJump > 0 && this.questionJump <= this.questions.length) {
        this.currentQuestionIndex = this.questionJump - 1;
        this.resetOption();
        this.saveQuiz();
        this.questionJump = null;
      }
    }
  },
  mounted() {
    this.fetchAndConvert()
  },
  computed: {
    currentQuestion() {
      return this.questions ? this.questions[this.currentQuestionIndex] : null;
    },
  },
}
</script>
<style>
@media (max-width:720px){
.custom-font{
  font-size: 1rem;
}
}
</style>