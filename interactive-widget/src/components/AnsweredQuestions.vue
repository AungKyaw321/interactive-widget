<template>
  <div>
    <h3>Answered Questions</h3>
    <ul>
      <li v-for="answer in answers" :key="answer.question.id">
        <strong>{{ answer.question.text }}:</strong> {{ answer.answer }}
        <button @click="$emit('edit', answer.question.id)">Edit</button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: ["answers"],
  emits: ["edit"],
};
</script>
