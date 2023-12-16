<template>
  <main class="mx-auto max-w-screen-xl text-center mt-2">
    <h1 class="text-blue-500 text-lg">Компилятор Фирсанов</h1>
    <div class="mt-8 flex flex-row items-start justify-between">
      <Input
        @lexical-update="lexicalAnalyzer"
        @syntax-update="syntaxAnalyzer"
        @semantic-update="semanticAnalyzer"
      />
      <Output
        :outputLexical="lexicalAnalyzerOutput"
        :outputSyntax="syntaxAnalyzerOutput"
        :outputSemantic="semanticAnalyzerOutput"
      />
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import Input from './components/Input.vue';
import Output from './components/Output.vue';

const lexicalAnalyzerOutput = ref({});
const syntaxAnalyzerOutput = ref({});
const semanticAnalyzerOutput = ref('');

function lexicalAnalyzer(output) {
  lexicalAnalyzerOutput.value = output;
}

function syntaxAnalyzer(output) {
  syntaxAnalyzerOutput.value = output;
}

function semanticAnalyzer(output) {
  semanticAnalyzerOutput.value = output;
}
</script>

<style scoped></style>
