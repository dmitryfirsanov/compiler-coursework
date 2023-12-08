<template>
  <main class="mx-auto max-w-screen-xl text-center mt-2">
    <h1 class="text-blue-500 text-lg">Компилятор Фирсанов</h1>
    <div class="mt-8 flex flex-row items-center justify-between">
      <Input @tokens-updated="updateTokens" />
      <Output :output-code="tokens" />
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import Input from './components/Input.vue';
import Output from './components/Output.vue';
const tokens = ref({
  keywords: [],
  identifiers: [],
  numbers: [],
  separators: [],
  unknown: [],
});

function updateTokens(newTokens) {
  console.log(newTokens)
  tokens.value = newTokens;
}
</script>

<style scoped></style>
