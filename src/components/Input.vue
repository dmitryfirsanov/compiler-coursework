<template>
  <div>
    <textarea
      v-model="inputCode"
      @input="syntaxAnalyzer"
      cols="60"
      rows="22"
      class="editor w-full p-10 text-sm outline-none text-gray-900 bg-white rounded border-0 dark:bg-gray-800 focus:ring-0 dark:text-white dark:placeholder-gray-400"
      placeholder="Write a code..."
    ></textarea>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import peggy from 'peggy';

const emits = defineEmits(['tokens-updated']);
const inputCode = ref('');
const tokens = ref({});

function lexicaдAnalyzer() {
  tokens.value = {
    keywords: [],
    identifiers: [],
    numbers: [],
    separators: [],
    unknown: [],
  };

  const code = inputCode.value;
  const globalRegex = /program|var|begin|end|integer|real|boolean|read|output|do\swhile|true|false|switch|case|for|let|[a-z]+\d*[a-z]*|\d+|[+\-*/=<>]|\(|\)|,|;|:|:=|!|&&|\|\||\.{1,2}|\{|\}/g;

  const matches = code.match(globalRegex);

  if (matches) {
    for (const match of matches) {
      tokens.value[getTokenType(match)].push(match);
    }
  }

  emits('tokens-updated', tokens);
}

function getTokenType(token) {
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
}

function syntaxAnalyzer() {
  const code = inputCode.value;

  const grammar = `start = программа

буква = [a-z]
цифра = [0-9]
_ "whitespace" = [ \\t\\n\\r]*

идентификатор = буква цифра* последовательностьБукв*
последовательностьБукв = буква цифра* последовательностьБукв?
число = цифра+

тип = "integer" / "real" / "boolean"

программа = "program" _ "var" _ описание _ "begin" _ оператор+ _ "end"

описание = тип _ идентификатор _ (',' _ идентификатор)*

тело = "begin" (оператор ';' )* "end"

оператор = присваивание / условный / фиксированныйЦикл / условныйЦикл / составной / ввод / вывод

присваивание = ("let" _)? идентификатор _ ":=" _ выражение ";"

условный = "switch" выражение "{" "case" идентификатор ":" оператор "}" 

фиксированныйЦикл = "for" "[" выражение ";" выражение ";" выражение "]" оператор

условныйЦикл = "do while" выражение оператор "loop"

составной = "begin" оператор+ "end"

ввод = "read" "(" идентификатор (',' идентификатор)* ")"

вывод = "output" "(" выражение (_ выражение)* ")"

выражение = сумма / произведение / логическаяКонстанта ";"

сумма = произведение ("+" / "-" / "&&") произведение*

произведение = множитель / ("*" / "/" / "||") множитель*

множитель = идентификатор / число / логическаяКонстанта / "!" множитель / "(" выражение ")"

логическаяКонстанта = "true" / "false"`;

  const parser = peggy.generate(grammar);
  try {
    const result = parser.parse(code);
    console.log(JSON.stringify(result, null, 2));
  } catch (error) {
    console.log(error);
  }
}
</script>
