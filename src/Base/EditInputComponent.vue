<template>
  <div>
    <edit-history-component :field="field" v-if="historyNeeded()"> </edit-history-component>

    <slot name="header"></slot>
    <div v-if="isInput">
      <ValidationProvider v-slot="{ errors }" :rules="validationRules" :name="inputField">
        <v-text-field
          :name="inputField"
          :id="inputField"
          :label="inputLabel"
          hide-details="auto"
          v-model="translated"
          :readonly="readonly"
          :error-messages="errors.api ? errors.api : errors"
          :color="!errors.api && translated ? 'success' : ''"
          :append-icon="suggestionAllowed ? 'fal fa-info-circle' : ''"
          @click:append="getSuggestion"
          @keyup="inputChanged()"
        >
        </v-text-field>
      </ValidationProvider>
    </div>

    <div v-if="isTextArea">
      <v-textarea
        :name="inputField"
        :id="inputField"
        filled
        :label="inputLabel"
        auto-grow
        v-model="translated"
        :readonly="readonly"
        :error-messages="apiErrors"
        :append-icon="suggestionAllowed ? 'fal fa-info-circle' : ''"
        @click:append="getSuggestion"
        @keyup="inputChanged()"
      ></v-textarea>
    </div>

    <div v-if="isHtml">
      <span v-if="suggestionAllowed">
        <a href="#"><span class="fal fa-info-circle" @click="getSuggestion"></span></a>
      </span>
      <quill-editor
        :name="inputField"
        :id="inputField"
        ref="myQuillEditor"
        v-model="translated"
        :options="editorOption"
        @change="onEditorChange($event)"
      />
      <!--      @blur="onEditorBlur($event)"-->
      <!--      @focus="onEditorFocus($event)"-->
      <!--      @ready="onEditorReady($event)"-->
      <v-checkbox
        v-model="autoSyncTxtWithHtml"
        class="mt-0"
        :label="$t('multilingual.misc.autosync_html_tTxt')"
        @change="clickedAutoSync"
      >
      </v-checkbox>
    </div>
  </div>
</template>

<script>
import { quillEditor } from 'vue-quill-editor';
import EventBus from '../Shared/event-bus';
// import 'quill/dist/quill.core.css'; /* eslint-disable-line */
// import 'quill/dist/quill.snow.css'; /* eslint-disable-line */
// import 'quill/dist/quill.bubble.css'; /* eslint-disable-line */

import store from '../Shared/store';
import EditHistoryComponent from './EditHistoryComponent.vue';

const htmlToText = require('html-to-text');

export default {
  components: {
    EditHistoryComponent,
    quillEditor,
  },

  props: {
    inputType: {
      type: String,
      default: 'INPUT',
    },

    inputLabel: {
      type: String,
      required: true,
    },

    inputField: {
      type: String,
      required: true,
    },

    inputData: {
      type: String,
      required: true,
    },

    apiErrors: {
      type: Array,
    },

    suggestionAllowed: {
      type: Boolean,
    },

    autoSync: {
      type: Boolean,
    },

    readonly: {
      type: Boolean,
    },

    validationRules: {
      type: String,
    },
  },

  data() {
    return {
      translated: '',
      state: store.state,

      editorOption: {
        // Some Quill options...
      },
      autoSyncTxtWithHtml: this.autoSync,

      errors: {
        api : this.apiErrors,
      }
    };
  },

  computed: {
    editor() {
      return this.$refs.myQuillEditor.quill;
    },

    isInput() {
      return this.inputType === 'INPUT';
    },

    isTextArea() {
      return this.inputType === 'TEXTAREA';
    },
    isHtml() {
      return this.inputType === 'HTML';
    },
  },

  mounted() {
    this.init();
  },

  watch: {
    meta() {
      this.init();
    },

    inputData() {
      this.translated = this.inputData;
    },
  },

  methods: {
    clickedAutoSync(event) {
      this.onEditorChange(event);
      this.$emit('update:autoSync', this.autoSyncTxtWithHtml);
    },

    historyNeeded() {
      return this.state.selectedItem.meta.langBase !== this.state.selectedItem.meta.langTranslation;
    },

    init() {
      this.translated = this.inputData;
    },

    inputChanged() {
      this.errors.api = {};
      this.$emit('update:inputData', this.translated);
      this.$emit('updated');
    },

    getSuggestion() {
      if (this.suggestionAllowed) {
        this.state.suggestion.field = this.field;
        EventBus.$emit('multilingual-suggestion-requested');
      }
    },

    // HTML Editor Methods
    // onEditorBlur(quill) {
    //   //
    // },
    // onEditorFocus(quill) {
    //   //
    // },
    // onEditorReady(quill) {
    //   //
    // },

    onEditorChange({ quill, html, text }) { /* eslint-disable-line */
      if (this.autoSyncTxtWithHtml) {
        const plain = htmlToText.fromString(this.translated, {
          wordwrap: 130,
          format: {
            // lineBreak(elem, fn, options) {
            //   const h = fn(elem.children, options);
            //   return '\r';
            // },
            anchor(elem, fn, options) {
              const h = fn(elem.children, options);
              const { href } = elem.attribs;
              const clickable = `<a href="${href}">${h}</a>`;
              return clickable;
              // return ` ${h} [${href}]`;
            },
          },
        });
        const fixedDoubleReturns = plain.replace(/\n\n/g, '\n');

        const truncateLength = 10;
        let resultPlainText = fixedDoubleReturns.substr(0, truncateLength);
        if (fixedDoubleReturns.length > truncateLength) {
          resultPlainText += '...';
        }

        this.$emit('updatedPlainText', fixedDoubleReturns, resultPlainText);
      }
      this.inputChanged();
    },
  },
};
</script>

<style>
ins {
  background: lightgreen;
  text-decoration: none;
}

del {
  background: pink;
}
</style>

<style scoped>
::v-deep .ql-editor {
  min-height: 260px !important;
}
</style>
