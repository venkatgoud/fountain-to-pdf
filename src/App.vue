<template>
  <div id="app">
    <section class="section has-background-grey-lighter">
      <div class="container">
        <h1 class="subtitle">Fountain to pdf</h1>
        <div class="field">
          <div id="input-file" class="file has-name is-info">
            <label class="file-label">
              <input
                class="file-input"
                type="file"
                name="inputfile"
                ref="inputFiles"
                @change="onFileChange"
              />
              <span class="file-cta">
                <span class="file-icon">
                  <i class="fas fa-upload"></i>
                </span>
                <span class="file-label">Choose a file…</span>
              </span>
              <span class="file-name">{{fileName}}</span>
            </label>
          </div>
        </div>
        <label class="checkbox">
          <input type="checkbox" id="indian_format" v-model="isIndian" />
          Indian Format
        </label>
      </div>
    </section>
    <PDFScreenplay :fountain="fountain" v-if="isFileLoaded" :indian="isIndian"/>
  </div>
</template>

<script>
import PDFScreenplay from "./components/PDFScreenplay.vue";

export default {
  name: "App",
  components: {
    PDFScreenplay
  },
  data: function() {
    return {
      fileName: "No file chosen",
      isIndian: false,
      isFileLoaded: false,
      fountain: ""
    };
  },
  methods: {
    onFileChange: function() {
      if (this.$refs.inputFiles.files.length > 0) {
        const file = this.$refs.inputFiles.files[0];
        this.fileName = file.name;
        var fileReader = new FileReader();
        const _instance = this;
        fileReader.onload = function(evt) {
          _instance.isFileLoaded = true;
          _instance.fountain = evt.target.result;
        };
        fileReader.readAsText(file);
      }
    }
  }
};
</script>