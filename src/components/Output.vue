<script setup>
import "../../node_modules/prismjs/themes/prism-solarizedlight.min.css";

defineProps({
    files: Object
})

/**
 * Copy the output to your clipboard
 * @param {*} e 
 */
function handleCopy(e) {
    const el = e.target;
    const initialText = el.innerText;
    el.innerText = "👍 Copied 👍"

    const idToCopy = el.dataset.copy;
    const codeToCopy = document.getElementById(idToCopy);
    navigator.clipboard.writeText(codeToCopy.innerText);

    setTimeout(() => {
        el.innerText = initialText
    }, 2000)
}
</script>

<template>
    <section class="output">
        <div class="downloads">
            <h2>Download your files</h2>
            <ol>
                <li v-for="file in files" :key="file.key">
                    <a :href="file.url" :download="file.name">{{ file.name }}</a>
                </li>
            </ol>
        </div>
        <div class="copy">
            <h2>...Or Copy Your HTML</h2>
            <div class="output--copy">
                <div class="output--single" v-for="file in files" :key="file.key">
                    <!-- <div class="output--html" v-html="file.html"></div> -->
                    <div class="output--code">
                        <h3>{{ file.name }}</h3>
                        <div class="highlight">
                            <button class="copy" @click="handleCopy" :data-copy="file.name">Copy</button>
                            <pre>
                                <code class="language-html" v-html="file.prism" :id="file.name"></code>
                            </pre>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </section>
</template>

<style scoped>
.output--copy {
    display: grid;
    gap: 1rem;
    grid-template-columns: repeat(auto-fit, minmax(20rem, 1fr));
    padding: 1rem;
}

.output--single {
    border: 1px solid currentColor;
    padding: 1rem;
    border-radius: 10px;
    block-size: max-content;
}

code[class*="language-"],
pre[class*="language-"],
pre {
    white-space: normal;
}

button.copy {
    border: 1px solid currentColor;
    border-radius: 0 0.15rem;
    cursor: pointer;
    float: right;
    font: inherit;
    margin-inline-start: 1rem;
    padding-block: 5px;
    padding-inline: 10px;
}
</style>