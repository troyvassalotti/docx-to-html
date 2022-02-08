<script setup>
import Converter from "./components/Converter.vue";
</script>

<template>
  <h1>NMG Word to HTML Converter</h1>
  <Converter />
</template>

<style>
*,
*::before,
*::after {
  box-sizing: inherit;
}

html {
  color-scheme: light dark;
}

html:focus-within {
  scroll-behavior: smooth;
}

body {
  margin: 0;
  padding: 0;
}
</style>