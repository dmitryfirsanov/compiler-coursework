<template>
  <div>
    <textarea
      v-model="inputCode"
      @input="parser"
      cols="65"
      rows="25"
      class="resize-none border rounded border-black p-3 bg-gray-100  outline-none"
      placeholder="Напиши код"
    ></textarea>
  </div>
</template>

<script>
import peg from 'pegjs';

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
      const globalRegex = /program|var|begin|end|integer|real|boolean|read|output|do\swhile|true|false|switch|case|for|let|[a-z]+\d*[a-z]*|\d+|[+\-*/=<>]|\(|\)|,|;|:|:=|!|&&|\|\||\.{1,2}|\{|\}/g;

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
        separator: /^[{}()=<>;:.,+\-*/!]$/,
      };

      if (regex.keywords.test(token)) {
        return 'keywords';
      } else if (regex.identifier.test(token)) {
        return 'identifiers';
      } else if (regex.number.test(token)) {
        return 'numbers';
      } else if (regex.separator.test(token) || token === "||" || token === "&&") {
        return 'separators';
      } else {
        return 'unknown';
      }
    },
    parser() {
      const code = this.outputCode;

      const grammar = `start = программа

буква = [a-z]
цифра = [0-9]
пробел = ' '

идентификатор = буква цифра* последовательность_букв*
последовательность_букв = буква цифра* последовательность_букв?
число = цифра+

ключевое_слово = "program" / "var" / "begin" / "end" / "integer" / "real" / "boolean" / "read" / "output" / "do while" / "true" / "false" / "switch" / "case" / "for" / "let"

разделитель = '(' / ')' / ',' / ';' / ':' / ':=' / '.' / '{' / '}' / '+' / '-' / '*' / '/' / '||' / '&&' / '!' / '=' / '<' / '>'

тип = "integer" / "real" / "boolean"

программа = "program" "var" описание ":begin" оператор+ "end."

описание = тип идентификатор (',' идентификатор)*

тело = "begin" (оператор ';' )* "end"

оператор = присваивание / условный / фиксированный_цикл / условный_цикл / составной / ввод / вывод

присваивание = ("let")? идентификатор ":=" выражение ";"

условный = "switch" выражение "{" "case" идентификатор ":" оператор "}" 

фиксированный_цикл = "for" "[" выражение ";" выражение ";" выражение "]" оператор

условный_цикл = "do while" выражение оператор "loop"

составной = "begin" оператор+ "end"

ввод = "read" "(" идентификатор (',' идентификатор)* ")"

вывод = "output" "(" выражение (пробел выражение)* ")"

выражение = сумма / произведение / логическая_константа ";"

сумма = произведение ("+" / "-" / "&&") произведение*

произведение = множитель / ("*" / "/" / "||") множитель*

множитель = идентификатор / число / логическая_константа / "!" множитель / "(" выражение ")"

логическая_константа = "true" / "false"
`;

      const parser = peg.generate(grammar);
      try {
        const result = parser.parse(this.inputCode.replace(/\s+/g, ''));
        console.log(JSON.stringify(result, null, 2));
      } catch (error) {
        console.log(error)
      }
    }
  },
};
</script>
