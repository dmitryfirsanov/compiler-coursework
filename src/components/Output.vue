<template>
  <div class="flex flex-col gap-2">
    <textarea
      name="output"
      cols="60"
      rows="6"
      class="editor w-full p-10 text-sm outline-none text-gray-900 bg-white rounded dark:bg-gray-800 focus:ring-0 dark:text-white dark:placeholder-gray-400"
      readonly
    >{{ outputLexical}}</textarea>
    <textarea
      name="output"
      cols="60"
      rows="6"
      class="editor w-full p-10 text-sm outline-none text-gray-900 bg-white rounded dark:bg-gray-800 focus:ring-0 dark:text-white dark:placeholder-gray-400"
      readonly
    >{{ outputSyntax }}</textarea>
    <textarea
      name="output"
      cols="60"
      class="editor w-full p-10 text-sm outline-none text-gray-900 bg-white rounded dark:bg-gray-800 focus:ring-0 dark:text-white dark:placeholder-gray-400"
      readonly
    >{{ outputSemantic }}</textarea>
  </div>
</template>

<script setup>
const props = defineProps({
  outputLexical: Object,
  outputSyntax: Object,
  outputSemantic: String,
})
</script>                                  