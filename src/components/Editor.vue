<style src='codemirror/lib/codemirror.css'></style>
<style src='codemirror/addon/hint/show-hint.css'></style>
<style src='lier/src/grammar/codemirror/theme.css'></style>

<template>
  <div class="editor">
    <textarea ref="editor" />
  </div>
</template>

<style lang="less" scoped>
.editor {
  border: solid #aaa 1px;  
}
</style>


<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';

import * as CodeMirror from 'codemirror';
import 'codemirror/addon/hint/show-hint';
import 'codemirror/mode/javascript/javascript';

import lierCodemirror from 'lier/src/grammar/codemirror';

CodeMirror.defineMode('lier', (config) => {
    return lierCodemirror.tokenizer;
});

const lierHint = lierCodemirror.autocomplete(CodeMirror);

@Component
export default class HelloWorld extends Vue {
  @Prop()
  public mode!: string;

  @Prop()
  public value!: string;

  private editor!: CodeMirror.EditorFromTextArea;

  protected mounted() {
    let theme = 'default';
    if (this.mode === 'lier') {
        theme = 'lier';
    }

    this.editor = CodeMirror.fromTextArea(
      this.$refs.editor as HTMLTextAreaElement,
      {
        mode: this.mode,
        lineNumbers: true,
        indentUnit: 4,
        theme,
        smartIndent: true,
        indentWithTabs: false,
        showCursorWhenSelecting: true,
        viewportMargin: Infinity,
      },
    );

    if (this.mode === 'lier') {
      this.editor.on('inputRead' as any, lierHint);
    }

    this.editor.on('change', (editor, e) => {
      this.$emit('change', editor.getValue());
    });

    this.editor.setValue(this.value);
  }

  protected beforeDestroy() {
    const a = 11;
    this.editor.getWrapperElement().remove();
  }
}
</script>
