<template>
  <div>
    <h1>Quiz App</h1>

    <div v-if="questions && questions.length > 0">
      <div>
        Question {{ currentQuestionIndex + 1 }} / {{ questions.length }}
      </div>
      <div v-if="currentQuestion">
        <div>
          {{currentQuestionIndex + 1}} . 
        </div> <div>{{ currentQuestion.Question }}</div>
        <div v-for="(option, index) in currentQuestion.Options" :key="index">
          <button v-on:click="checkAnswer(option)" :style="buttonStyle(option)">
            {{ option }}
          </button>
        </div>
      </div>
    </div>

    <div v-if="questions">
      <button @click="previousPage" :disabled="currentQuestionIndex === 0">Previous</button>
      <button @click="nextPage" :disabled="currentQuestionIndex > questions.length - 1">Next</button>
    </div>

    <div v-if="showResults">
      <div>Quiz Results </div>
      <div>score {{ score }} / {{ questions.length }}</div>
      </div>
    </div>

    <div v-if="currentQuestion && currentQuestion.Exam" style="background-color: yellow;">
       {{ questions.Exam }}
    </div>
    <div v-if="currentQuestion && currentQuestion.Explanation">
     {{ questions.Explanation }}
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
    };
  },
  methods: {
    async fetchAndConvert() {
      const csvUrl = "https://gist.githubusercontent.com/VishalBty/780fa362a5be0770ebdfa05f584ed039/raw";
      try {
        const response = await axios.get(csvUrl);

        const jsonObj = await csv().fromString(response.data)

        this.questions = jsonObj.map(({ Question, A, B, C, D, Answer , Exam , Explanation }) => ({
          Question: this.removeNumbers(Question),
          Exam ,
          Explanation,
          Options: [A, B, C, D],
          CorrectAnswer: Answer
        }));


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
    removeNumbers(question){
       return question.replace(/^\d+\.\s*/, '');
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
