<template>
  <h1>RichTextからMarkdownへの変換ツール</h1>
  <div id="app" class="container">
    <div class="editor-container">
      <p>【入力】RichText形式</p>
      <div class="quill-editor">
        <div ref="quillEditor"></div>
      </div>
    </div>
    <div class="markdown-container">
      <div class="output-header">
        <p class="output-message">【出力】Markdown形式</p>
        <button @click="copyToClipboard">コピー</button>
      </div>
      <div>
        <pre>{{ markdown }}</pre>
      </div>
    </div>
  </div>
  <div class="footer">
    <input type="checkbox" v-model="isNewLineAsParagraph" id="isNewLineAsParagraph" @change="richtextToMarkdown">
    <label for="isNewLineAsParagraph">改行を段落区切りとして扱う</label>
    <input type="checkbox" v-model="isIgnoreBoldInHeader" id="isIgnoreBoldInHeader" @change="richtextToMarkdown">
    <label for="isIgnoreBoldInHeader">見出し内の太字を無視する</label>
  </div>
</template>

<script lang="ts" setup>
import { ref, Ref, onMounted } from 'vue';
import Quill from 'quill';
import TurndownService from 'turndown';

const quillEditor: Ref<HTMLElement | null> = ref(null);
const markdown = ref('');
let quill: Quill | null = null;
let isNewLineAsParagraph = ref(false);
let isIgnoreBoldInHeader = ref(true);

const copyToClipboard = () => {
  navigator.clipboard.writeText(markdown.value)
    .then(() => {
      alert('Markdownをコピーしました');
    })
    .catch(err => {
      console.error('コピーに失敗しました: ', err);
      alert('コピーに失敗しました');
    });
};

const richtextToMarkdown = () => {
  const text_html = quill!.root.innerHTML;
  // https://github.com/mixmark-io/turndown?tab=readme-ov-file
  const turndownService = new TurndownService({
    headingStyle: 'atx',
    codeBlockStyle: 'fenced',
    hr: "---",
    bulletListMarker: "-",
  });

  let text_markdown = turndownService.turndown(text_html);
  if (!isNewLineAsParagraph.value) {
    text_markdown = text_markdown.replace(/\n\n/g, '\n');
  }
  if (isIgnoreBoldInHeader.value) {
    text_markdown = text_markdown.replace(/^(#+) \*\*(.+?)\*\*$/gm, '$1 $2');
  }
  markdown.value = text_markdown;
}

onMounted(() => {
  if (!quillEditor.value) {
    return;
  }
  quill = new Quill(quillEditor.value, {
    theme: 'snow',
  });
  quill.on('text-change', () => {
    richtextToMarkdown();
  });
});
</script>

<style>
.container {
  display: flex;
  justify-content: space-around;
  padding: 20px;
  text-align: left;
}
.editor-container,
.markdown-container {
  width: 45%;
  height: 70vh;
  border: 1px solid #ccc;
  padding: 10px;
  overflow-y: auto;
}
.quill-editor {
  height: 80%;
}
.output-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.output-message {
  margin: auto;
}
pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  text-align: left;
}
</style>
