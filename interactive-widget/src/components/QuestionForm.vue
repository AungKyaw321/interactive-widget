<template>
  <div class="question-form">
    <h3>Answer the following questions:</h3>
    <div v-for="question in questions" :key="question.id">
      <p>{{ question.text }}</p>

      <div v-if="question.type === 'text-enum'">
        <button
          v-for="option in question.options"
          :key="option"
          @click="submitAnswer(question, option)"
        >
          {{ option }}
        </button>
      </div>

      <div v-else-if="question.type === 'image-enum'">
        <img
          v-for="option in question.options"
          :key="option"
          :src="option"
          @click="submitAnswer(question, option)"
          class="image-option"
        />
      </div>

      <input
        v-else-if="question.type === 'text'"
        type="text"
        v-model="answers[question.id]"
        @blur="submitAnswer(question, answers[question.id])"
      />

      <input
        v-else-if="question.type === 'number'"
        type="number"
        v-model="answers[question.id]"
        @blur="submitAnswer(question, answers[question.id])"
      />
    </div>
  </div>
</template>

<script>
export default {
  props: ["questions"],
  emits: ["answer"],
  setup(props, { emit }) {
    const answers = {};

    function submitAnswer(question, answer) {
      if (answer) {
        emit("answer", { question, answer });
      }
    }

    return { answers, submitAnswer };
  },
};
</script>

<style scoped>
.image-option {
  width: 80px;
  cursor: pointer;
}
</style>
