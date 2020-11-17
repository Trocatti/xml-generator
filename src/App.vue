<style lang="scss">
@import "node_modules/bootstrap/scss/bootstrap";
@import "node_modules/bootstrap-vue/src/index.scss";

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html,
body,
#app {
  background-color: aliceblue;
}

.form-xml {
  .text-muted {
    color: red !important;
    &:focus {
      border: 0;
      outline: none;
    }
  }
}
</style>

<template>
  <div id="app" class="container">
    <b-card class="m-4" header="Form Data">
      <b-form
        @submit.prevent="onSubmit"
        @reset.prevent="onReset"
        class="form-xml"
      >
        <b-form-group id="input-group-1" label="File:" label-for="input-1">
          <b-form-file
            id="input-1"
            v-model="file"
            :state="Boolean(file)"
            placeholder="Choose a file or drop it here..."
            drop-placeholder="Drop file here..."
            accept=".667"
          ></b-form-file>
          <div class="mt-3">Selected file: {{ file ? file.name : "" }}</div>
        </b-form-group>

        <b-form-group id="input-group-2" label="Tag:" label-for="input-2">
          <b-form-input
            id="input-2"
            v-model="form.tag"
            @change="onChangeTag($event)"
            type="text"
            required
            placeholder="Informe o nome da tag, exemplo: cd_Uni_Origem."
          ></b-form-input>
        </b-form-group>

        <b-form-group
          id="input-group-3"
          label="Values:"
          label-for="input-3"
          description="Insira somente 10 valores possíveis separados por vírgula."
        >
          <b-form-textarea
            id="input-3"
            v-model="form.values"
            type="text"
            required
            :placeholder="
              'Informe os possíveis valores do campo: ' + getNameTag
            "
            rows="3"
            max-rows="6"
          ></b-form-textarea>
        </b-form-group>

        <b-form-group id="input-group-4" label="Name Zip:" label-for="input-4">
          <b-form-input
            id="input-4"
            v-model="form.name"
            type="text"
            required
            placeholder="Informe o nome do arquivo zip."
          ></b-form-input>
        </b-form-group>

        <div class="d-flex justify-content-end">
          <b-button class="mr-2" type="submit" variant="primary"
            >Gerar</b-button
          >
          <b-button type="reset" variant="danger">Limpar</b-button>
        </div>
      </b-form>
    </b-card>
  </div>
</template>

<script>
import _ from "lodash";
import FileSaver from "file-saver";
import JSZip from "jszip";
import x2js from "x2js";

const X2JS = new x2js();

const TAG_DEFAULT = "ptuA550.Tipo_Questionamento.Quest_Reembolso.idReembolso";

import {
  BForm,
  BFormGroup,
  BFormInput,
  BFormTextarea,
  BFormFile,
  BButton,
  BCard
} from "bootstrap-vue";

export default {
  components: {
    BButton,
    BForm,
    BFormGroup,
    BFormInput,
    BFormTextarea,
    BFormFile,
    BCard
  },

  data() {
    return {
      file: null,
      form: {
        tag: TAG_DEFAULT,
        values: "",
        name: ""
      }
    };
  },

  computed: {
    getNameTag() {
      return this.form.tag.split(".").pop();
    }
  },

  methods: {
    async download(nameFile, nameZip, xml) {
      try {
        const zip = new JSZip();
        zip.file(nameFile, xml);
        const content = await zip.generateAsync({ type: "blob" });
        if (content) {
          FileSaver.saveAs(content, nameZip);
        }
      } catch (error) {
        console.error(error);
      }
    },

    parseXML(data) {
      const xmlAsObj = X2JS.js2xml(data);
      const header = `<?xml version="1.0" encoding="iso-8859-1"?>`;
      return header.concat(xmlAsObj);
    },

    generation(file, tagName, values) {
      try {
        const fr = new FileReader();
        fr.readAsText(file);
        fr.onload = () => {
          const obj = fr.result;

          const parseString = X2JS.xml2js(obj);

          values.forEach(async (val, key) => {
            const newObj = JSON.parse(JSON.stringify(parseString));

            const update = _.update(newObj, tagName, function(n) {
              n.__text = val;
              return n;
            });

            const xml = this.parseXML(update);

            const nameFile = this.file.name;
            const nameZip = `${key + 1}_${this.form.name}_${val}`;
            await this.download(nameFile, nameZip, xml);
          });
        };
      } catch (error) {
        console.error(error);
      }
    },

    onSubmit() {
      const values = this.form.values.split(",") || [];
      this.generation(this.file, this.form.tag, values);
    },

    onReset() {
      this.file = null;
      this.form = {
        tag: TAG_DEFAULT,
        values: "",
        name: ""
      };
    },

    onChangeTag() {
      this.form.name = this.getNameTag;
    }
  },

  mounted() {
    this.form.name = this.getNameTag;
  }
};
</script>
