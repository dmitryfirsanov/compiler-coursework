<template>
  <div>
    <textarea
      v-model="inputCode"
      @input="analyzers"
      cols="60"
      rows="23"
      class="editor w-full h-full p-10 text-sm outline-none text-gray-900 bg-white rounded border-0 dark:bg-gray-800 focus:ring-0 dark:text-white dark:placeholder-gray-400"
      placeholder="Write a code..."
    ></textarea>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import peggy from 'peggy';

const emits = defineEmits(['lexical-update', 'syntax-update', 'semantic-update']);

const inputCode = ref('');
const lexicalAnalyzerOutput = ref({});
const syntaxAnalyzerOutput = ref({});
const semanticAnalyzerOutput = ref('');

function analyzers() {
  lexicalAnalyzerOutput.value = lexicalAnalyzer();
  emits('lexical-update', lexicalAnalyzerOutput.value);
  syntaxAnalyzerOutput.value = syntaxAnalyzer();
  emits('syntax-update', syntaxAnalyzerOutput.value);
  semanticAnalyzerOutput.value = semanticAnalyzer();
  emits('semantic-update', semanticAnalyzerOutput.value);
}

function lexicalAnalyzer() {
  const code = inputCode.value;

  const result = {
    keywords: [],
    identifiers: [],
    numbers: [],
    separators: [],
    unknown: [],
  };

  const globalRegex = /program|var|begin|end|integer|real|boolean|read|output|do\swhile|true|false|switch|case|for|let|[a-z]+\d*[a-z]*|\d+|[+\-*/=<>]|\(|\)|,|;|:|:=|!|&&|\|\||\.{1,2}|\{|\}/g;

  const matches = code.match(globalRegex);

  if (matches) {
    for (const match of matches) {
      result[getTokenType(match)].push(match);
    }
  }

  return result;
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

условный = "switch" _ идентификатор _ "{" (_ "case" _ идентификатор _ ":" _ оператор)* "}" 

фиксированныйЦикл = "for" "(" выражение ";" _ выражение ";" _ выражение ")" _ "{" оператор "}"

условныйЦикл = "do while" _ выражение _ оператор _ "loop"

составной = "begin" оператор+ "end"

ввод = "read" "(" идентификатор (',' идентификатор)* ")"

вывод = "output" "(" выражение (_ выражение)* ")"

выражение = сумма / произведение / логическаяКонстанта ";"

сумма = произведение ("+" / "-" / "&&") произведение*

произведение = множитель / ("*" / "/" / "||") множитель*

множитель = идентификатор / число / логическаяКонстанта / "!" множитель / "(" выражение ")"

логическаяКонстанта = "true" / "false"`;

  const parser = peggy.generate(grammar);

  let result;
  try {
    result = parser.parse(code);
  } catch (error) {
    result = error;
  }

  return result
}

function semanticAnalyzer() {
  const code = inputCode.value;

  const program = code.split(/\s+/);
  
  // Проверка начала программы
  if (program[0] !== "program") {
    return "Ошибка: отсутствует ключевое слово 'program'";
  }

  // Проверка блока переменных
  if (program[1] !== "var") {
    return "Ошибка: отсутствует ключевое слово 'var'";
  }

  if (program[2] !== "integer") {
    if (program[2] !== "real")
      if (program[2] === "bollean")
        return "Ошибка: отсутствует блок описания";
  }

  // Проверка блока операторов
  let beginIndex = program.indexOf("begin");
  if (beginIndex === -1) {
    return "Ошибка: отсутствует ключевое слово 'begin'";
  }
  let operatorsBlock = program.slice(beginIndex + 1, program.length - 1);
  if (operatorsBlock.length === 0) {
    return "Ошибка: блок операторов пуст";
  }

  if (program[program.length - 1] !== "end") {
    return "Ошибка: отсутствует ключевое слово 'end'";
  }

  let variables = program.slice(program.indexOf("let") + 1, beginIndex - 1);
  for (let variable of variables) {
    if (!isValidIdentifier(variable)) {
      return `Ошибка: некорректный идентификатор '${variable}'`;
    }
  }

  variables = program.slice(program.indexOf("integer") + 1, beginIndex - 1);
  for (let variable of variables) {
    if (!isValidIdentifier(variable)) {
      return `Ошибка: некорректный идентификатор '${variable}'`;
    }
  }

  variables = program.slice(program.indexOf("real") + 1, beginIndex - 1);
  for (let variable of variables) {
    if (!isValidIdentifier(variable)) {
      return `Ошибка: некорректный идентификатор '${variable}'`;
    }
  }

  variables = program.slice(program.indexOf("boolean") + 1, beginIndex - 1);
  for (let variable of variables) {
    if (!isValidIdentifier(variable)) {
      return `Ошибка: некорректный идентификатор '${variable}'`;
    }
  }

  let ifIndex = program.indexOf("if");
  if (ifIndex !== -1) {
    let condition = program.slice(ifIndex + 1, program.indexOf("then"));
    if (condition.length === 0) {
      return "Ошибка: отсутствует условие в операторе if";
    }
    let thenIndex = program.indexOf("then");
    let ifBlock = program.slice(thenIndex + 1, program.indexOf("else") || program.indexOf("end"));
    if (ifBlock.length === 0) {
      return "Ошибка: отсутствует блок операторов после then в операторе if";
    }
    // Дополнительные проверки для условного цикла могут быть добавлены здесь
  }

  // Проверка фиксированного цикла
  let forIndex = program.indexOf("for");
  if (forIndex !== -1) {
    let loopVariable = program[forIndex + 1];
    let fromIndex = program.indexOf("from");
    let toIndex = program.indexOf("to");
    let loopBlock = program.slice(toIndex + 1, program.indexOf("do"));
    if (!isValidIdentifier(loopVariable)) {
      return `Ошибка: некорректный идентификатор '${loopVariable}' в операторе for;`
    }
    if (fromIndex === -1 || toIndex === -1 || loopBlock.length === 0) {
      return "Ошибка: некорректная структура оператора for";
    }
    // Дополнительные проверки для фиксированного цикла могут быть добавлены здесь
  }

  if (program.find(element => element === "(*")) {
    let commentOver = program.find(element => element === "*)")
    if (!commentOver) return "Ошибка: комментарий не был закрыт"
  }

  // Проверка ввода
  let readIndex = program.indexOf("read");
  if (readIndex !== -1) {
    let inputVariables = program.slice(readIndex + 1, program.indexOf(")"));
    for (let variable of inputVariables) {
      if (!isValidIdentifier(variable)) {
        return `Ошибка: некорректный идентификатор '${variable}' в операторе read`;
      }
    }
  }

  // Проверка вывода
  let outputIndex = program.indexOf("output");
  if (outputIndex !== -1) {
    let outputExpressions = program.slice(outputIndex + 1, program.indexOf(")"));
    for (let expression of outputExpressions) {
      if (!isValidExpression(expression)) {
        return `Ошибка: некорректное выражение '${expression}' в операторе output`;
      }
    }
  }

  let switchIndex = program.indexOf("switch");
  if (switchIndex !== -1) {
    let identifier = program[switchIndex + 1];
    if (!isValidIdentifier(identifier)) {
      return `Ошибка: некорректный идентификатор'${identifier}' в операторе switch`;
    }
  }

  return "Программа корректна";
}

function isValidIdentifier(identifier) {
  let identifierRegex = /^[a-zA-Z_][a-zA-Z0-9_]*$/;
  return identifierRegex.test(identifier);
}
</script>
