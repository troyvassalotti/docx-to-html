<script setup>
import nodePandoc from "node-pandoc"
import {reactive} from "vue"

const store = reactive({
  downloads: []
})

function runPandoc(el) {
  const fileList = el.files

  for (const file of fileList) {
    let index = 0
    store.downloads.push({
      name: file.name,
      url: URL.createObjectURL(file),
      key: index
    })
    index++
    console.log(store.downloads);
  }
}
</script>

<template>
  <h1>Pandoc</h1>
  <form enctype="multipart/form-data" novalidate @submit="preventDefault()">
    <label for="upload">Upload Your Docx <input id="upload" type="file" multiple accept=".doc, .docx" @change="runPandoc($event.target)"></label>
  </form>
  <div class="output" v-if="store.downloads.length > 0">
    <h2>Download your files</h2>
    <a v-for="file in store.downloads" :key="file.key" :href="file.url" :download="file.name">Download</a>
  </div>
</template>

<style scoped>
a {
  color: #42b983;
}
</style>
