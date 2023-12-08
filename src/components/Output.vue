<template>
  <div>
    <textarea
      name="output"
      cols="60"
      rows="22"
      class="editor w-full p-10 text-sm outline-none text-gray-900 bg-white rounded dark:bg-gray-800 focus:ring-0 dark:text-white dark:placeholder-gray-400"
      readonly
    >{{ outputCode }}</textarea>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  outputCode: Object,
})

const displayTokens = computed(() => {
  if (props.outputCode.value) {
    return Object.entries(props.outputCode.value).map(([key, value]) => {
      return `${key} : ${value};`;
    }).join('\n');
  } else {
    return '';
  }
})

// const displayPegResult = computed(() => {
  
// })
</script>                                  