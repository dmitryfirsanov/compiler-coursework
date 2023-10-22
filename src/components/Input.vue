<template>
  <div>
    <textarea
      v-model="inputCode"
      @input="analyzeCode"
      cols="65"
      rows="25"
      class="resize-none border rounded border-black p-3 bg-gray-100  outline-none"
      placeholder="Напиши код"
    ></textarea>
  </div>
</template>

<script>
export default {
  data() {
    return {
      inputCode: '',
      tokens: {},
    };
  },
  methods: {
    analyzeCode() {
      this.tokens = {
        keywords: [],
        identifiers: [],
        numbers: [],
        separators: [],
        unknown: [],
      };

      const code = this.inputCode;
      const globalRegex = /program|var|begin|end|integer|real|boolean|read|output|do\swhile|true|false|switch|case|for|let|[a-z]+\d*[a-z]*|\d+|[+\-*/=<>]|\(|\)|,|;|:|:=|\.{1,2}|\{|\}/g;

      const matches = code.match(globalRegex);

      if (matches) {
        for (const match of matches) {
          this.tokens[this.getTokenType(match)].push(match);
        }
      }

      this.$emit('tokens-updated', this.tokens);
    },
    getTokenType(token) {
      const regex = {
        keywords: /^program|var|begin|end|integer|real|boolean|read|output|do\swhile|true|false|switch|case|for|let$/,
        identifier: /^[a-z]\w*$/,
        number: /^\d+$/,
        separator: /^[{}()=<>;:.,+\-*/]$/,
      };

      if (regex.keywords.test(token)) {
        return 'keywords';
      } else if (regex.identifier.test(token)) {
        return 'identifiers';
      } else if (regex.number.test(token)) {
        return 'numbers';
      } else if (regex.separator.test(token)) {
        return 'separators';
      } else {
        return 'unknown';
      }
    },
  },
};
</script>
