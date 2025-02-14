<template>
  <div>
    <h3>Was this solution helpful?</h3>
    <button @click="$emit('feedback', 'yes')">Yes</button>
    <button @click="$emit('feedback', 'no')">No</button>
  </div>
</template>
