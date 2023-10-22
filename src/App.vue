<template>
  <main class="mx-auto max-w-screen-xl">

    <div class="mt-8 flex flex-row items-center justify-between">
      <Input @tokens-updated="updateTokens" />

      <h3 class="text-blue-500 text-lg">Лексический анализ</h3>

      <Output :output-code="tokens" />
    </div>
  </main>
</template>

<script>
import Input from './components/Input.vue';
import Output from './components/Output.vue';

export default {
  components: { Input, Output },
  data() {
    return {
      tokens: {},
    };
  },
  methods: {
    updateTokens(tokens) {
      this.tokens = tokens;
    },
  },
}
</script>

<style scoped></style>
