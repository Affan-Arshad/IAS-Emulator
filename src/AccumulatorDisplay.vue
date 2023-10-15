<template>
  <div class="accumulator-display">
    Accumulator: {{ value }}
  </div>
</template>

<script>
export default {
  props: {
    value: Number,
  },
};
</script>