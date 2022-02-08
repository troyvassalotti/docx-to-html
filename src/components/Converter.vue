<script setup>
import mammoth from "mammoth/mammoth.browser";
import Prism from "prismjs"
import { reactive } from "vue";

/**
 * Store files ready to be downloaded in state
 * @type {Array}
 */
const store = reactive({
  downloads: [],
});

/**
 * Create an array buffer for mammoth to use
 * @link https://mike.zwobble.org/projects/mammoth/
 */
function readFileAsArrayBuffer(input, callback) {
  const file = input;
  const reader = new FileReader();

  reader.onload = (event) => {
    const arrayBuffer = event.target.result;
    callback(arrayBuffer);
  };

  reader.readAsArrayBuffer(file);
}

function highlightOutput(code) {
  const html = Prism.highlight(code, Prism.languages.html, "html")
  return html
}

function handleFileSelect(el) {
  const fileList = el.files;
  let index = 0;

  /**
   * Replace default image conversion with a blank element and custom attributes
   * Default functionality is commented out. It returns a base64 URI of the image.
   */
  const options = {
    convertImage: mammoth.images.imgElement(function (image) {
      // return image.read("base64").then(function (imageBuffer) {
      //   return {
      //     src: "data:" + image.contentType + ";base64," + imageBuffer,
      //   };
      // });
      return image.read("base64").then((imageBuffer) => {
        return {
          src: "#external-image",
          alt: "",
          loading: "lazy",
          decoding: "async",
        };
      });
    }),
  };

  for (const file of fileList) {
    const outputName = file.name.replace(/\.[^.]*$/, "");

    readFileAsArrayBuffer(file, (arrayBuffer) => {
      mammoth
        .convertToHtml({ arrayBuffer: arrayBuffer }, options)
        .then((result) => {
          const html = result.value
            .replace(/\u00a0/gi, " ")
            .replace(/&nbsp;/gi, " ")
            .replace(/\u2019/gi, "'")
            .replace(/\u2013/gi, "&mdash;");
          const messages = result.messages;
          return { html, messages };
        })
        .then((result) => {
          const html = result.html
          const output = new Blob([result.html], { type: "text/plain" });
          const outputUrl = URL.createObjectURL(output);
          console.log(result.messages);

          return {outputUrl, html};
        })
        .then((result) => {
          store.downloads.push({
            name: `${outputName}.html`,
            url: result.outputUrl,
            html: result.html,
            key: index,
          });
        });
    });
    index++;
  }
}
</script>

<template>
  <main>
    <form enctype="multipart/form-data" novalidate @submit="preventDefault()">
      <label for="upload"
        >Upload Your Docx
        <input
          id="upload"
          type="file"
          multiple
          accept=".doc, .docx"
          @change="handleFileSelect($event.target)"
      /></label>
    </form>
    <div class="output" v-if="store.downloads.length > 0">
      <h2>Download your files</h2>
      <ol>
        <li v-for="file in store.downloads" :key="file.key">
          <a :href="file.url" :download="file.name">{{ file.name }}</a>
        </li>
      </ol>
      <h2>...Or Copy Your HTML</h2>
      <div class="output--html">
        <pre v-for="file in store.downloads" :key="file.key">
          <code>{{ file.html }}</code>
        </pre>
      </div>
    </div>
  </main>
</template>

<style scoped>
.output--html {
  display: flex;
}

pre {
    white-space: normal;
}
</style>
