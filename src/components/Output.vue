<template>
  <div>
    <textarea
      name="output"
      cols="65"
      rows="25"
      class="resize-none border rounded border-black p-3 bg-gray-100 outline-none cursor-default"
      readonly
    >{{ displayTokens }}</textarea>
  </div>
</template>

<script>

export default {
  props: {
    outputCode: Object,
  },
  computed: {
    displayTokens() {
      return Object.entries(this.outputCode).map(([key, value]) => {
        return `${key} : [${value.join(", ")}]`
      });
    },
  },
}
</script>