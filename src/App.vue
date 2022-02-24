<script setup>
import Converter from "./components/Converter.vue";
</script>

<template>
  <h1>Word to HTML Converter</h1>
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
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  margin: 0;
  padding: 0;
}

#app {
  inline-size: 95%;
  margin-inline: auto;
  max-inline-size: 1600px;
}
</style>
