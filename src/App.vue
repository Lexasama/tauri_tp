<template>
  <div class="container">
    <div style="display:inline-block">
      <h1>Folders and files</h1>

      <input
        type="text"
        name="folder"
        id="folder"
        v-model="path"
        placeholder="Enter a path"
      />
      <button @click="read">search</button>

      <ul>
        <li v-for="f in files" :key="f" @click="goto(f)" :style="f.children == undefined ? 'color:green' : 'color:red'">
          {{ f.children == undefined ? "file" : "Folder: " }}
          {{ f.name }}
        </li>
      </ul>
    </div>

    <div style="display:inline-block">
      <MonacoEditor
        height="600"
        width="600"
        language="json"
        :code="code"
        :editorOptions="options"
        @mounted="onMounted"
        @codeChange="onCodeChange"
      >
      </MonacoEditor>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { fs, path } from "@tauri-apps/api";
import MonacoEditor from "vue-monaco-editor";

export default defineComponent({
  name: "App",
  components: {
    MonacoEditor,
  },
  data() {
    let files: Array<any> = [];
    let editor: any;
    return {
      code: "",
      path: "./",
      files,
      editor,
      options: {
        selectOnLineNumbers: false,
      },
    };
  },
  mounted() {
    path.homeDir().then((v) => {
      this.path = v;
    this.read();
   });
   
  },
  methods: {
    onMounted(editor: any) {
      this.editor = editor;
    },

    goto(f: fs.FileEntry) {
      if (f.children == undefined) {
        // this.file = f;
        this.readFile(f.path);
        return;
      }
      this.path = f.path;
      this.read();
    },
    read() {
      this.files = [];
      fs.readDir(this.path).then((d) => {
        d.forEach((c) => {
          this.files.push(c);
        });
      });
    },

    readFile(path: string) {
      fs.readTextFile(path).then((txt) => {
        console.log(txt);
        this.editor.setValue(txt);
      });
    },
    onChange(value: any) {
      console.log(value);
    },
    onCodeChange(editor: any) {
      console.log(this.editor.getValue());
    },
  },
});
</script>

<style lang="scss">
.container {
  display: flex;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
