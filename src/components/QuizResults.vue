<template>
    <!-- Results -->
    <div class="d-flex flex-column justify-content-center align-items-center vh-100">
      <div class="text-center text-white">
        <div class="display-1">Quiz Results</div>
        <div class="fs-1">Score: {{ score }} / {{ totalQuestions }}</div>
      </div>
    </div>
</template>
<script>
export default{
    props : [ 'score','totalQuestions']
}

</script>