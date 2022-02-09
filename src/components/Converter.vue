<script setup>
import mammoth from "mammoth/mammoth.browser";
import Prism from "prismjs";
import { reactive } from "vue";
import Output from "./Output.vue";

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

function handleFileSelect(el) {
  const fileList = el.files;
  let index = 0;

  /**
   * Replace default image conversion with a blank element and custom attributes.
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
          /**
           * Replace invisible characters in the output.
           */
          const html = result.value
            // Replace empty spaces before closing tags
            .replace(/\s<\/strong><\/p>/gi, "</strong></p>")
            .replace(/\s<\/em><\/p>/gi, "</em></p>")
            .replace(/\s<\/p>/gi, "</p>")
            .replace(/\s<\/em><\/strong><\/p>/gi, "</em></strong></p>")
            .replace(/\s<\/strong><\/em><\/p>/gi, "</strong></em></p>")

            // Remove empty tags
            // Purposefully run these multiple times to cover times where removing the first reveals the second
            .replace(/<[a-z]*>\s?<\/[a-z]*>/gi, "")
            .replace(/<[a-z]*>\s?<\/[a-z]*>/gi, "")
            .replace(/<[a-z]*>\s?<\/[a-z]*>/gi, "")

            // Replace invisible characters
            .replace(/\u00a0/gi, " ")
            .replace(/&nbsp;/gi, " ")
            .replace(/\u2019/gi, "'")
            .replace(/“/gi, '"')
            .replace(/”/gi, '"')
            .replace(/\u2013/gi, "&mdash;");
          const messages = result.messages;
          return { html, messages };
        })
        .then((result) => {
          const html = result.html;
          /**
           * Create a Blob of the output, in order to create an Object URL, which can then be downloaded in the browser.
           */
          const output = new Blob([result.html], { type: "text/plain" });
          const outputUrl = URL.createObjectURL(output);
          console.log(result.messages);

          return { outputUrl, html };
        })
        .then((result) => {
          store.downloads.push({
            name: `${outputName}.html`,
            url: result.outputUrl,
            html: result.html,
            prism: Prism.highlight(result.html, Prism.languages.html, "html"),
            key: index,
          });
        });
    });
    index++;
  }
}
</script>

<template>
  <main class="converter">
    <form enctype="multipart/form-data" novalidate @submit="preventDefault()">
      <label for="upload">Upload your .docx file(s):</label>
      <input
        id="upload"
        type="file"
        multiple
        accept=".doc, .docx"
        @change="handleFileSelect($event.target)"
      />
    </form>
    <Output v-if="store.downloads.length > 0" :files="store.downloads" />
  </main>
</template>
