<template>
  <div>
    <h1>Student Scores Tracking is: {{ totalScore }}</h1>
    <h1>The average scores is: {{ average }}</h1>
    <h1>Status: {{ examStatus }}</h1>
  </div>
</template>

<script setup>
import { ref, computed, watch } from "vue";
const studentName = ref("Ms.Sreyroth");
const score = ref([
  { subject: "English", score: 100 },
  { subject: "Vue", score: 180 },
  { subject: "Laravel", score: 80 },
]);

const passMark = ref(50);
let totalScore = computed(() => {
  let total = 0;
  score.value.forEach((item) => {
    total += item.score;
  });
  return total;
});

const average = computed(() => {
  return totalScore.value === 0 ? 0 : totalScore.value / score.value.length;
});

const examStatus = computed(() => {
  if (average.value >= passMark.value) {
    return "Passed";
  } else {
    return "Failed";
  }
});

watch(
  score,
  (newValue) => {
    localStorage.setItem("score", JSON.stringify(newValue));
  },
  { deep: true },
);
</script>

<style lang="scss" scoped></style>
