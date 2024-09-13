<template>
    <div v-if="questions" class="d-flex gap-4 pt-3 pb-3">
        <button class=" btn btn-dark " @click="previousPage" :disabled="currentQuestionIndex === 0">Previous</button>
        <button class=" btn btn-success" @click="nextPage"
          :disabled="currentQuestionIndex > questions.length - 1">Next</button>
      </div>
</template>
<script>
 export default {
    props : []
 }
</script>