<template>
  <div>
    <CodeInput
      :loading="form.isSubmitting"
      :invalid-input="confirmCodeInvalid"
      ref="codeInputComponent"
      class="input"
      type="number"
      :fields="6"
      @change="onCodeChange"
      @complete="onCodeComplete"
      @enter="onCodeEnter"
    />

    <div class="form-horizontal">
      <div class="form-group">
        <div v-if="errorMessage">
          <div class="alert alert-danger" style="overflow-wrap: break-word">
            <strong>{{ errorMessage }}</strong>
          </div>
        </div>

        <div v-if="successMessage">
          <div class="alert alert-success" style="overflow-wrap: break-word">
            <strong>{{ successMessage }}</strong>
          </div>
        </div>
      </div>

      <div class="form-group">
        <div class="row">
          <div class="col-6">
            <button
              id="authenticator-cancel-button"
              class="btn btn-danger"
              @click="cancelAuthenticator"
            >
              {{ cancelBtnLabel }}
            </button>
          </div>

          <div class="col-6">
            <button-submit
              :is-loading="submitting"
              :disabled="confirmCodeInvalid"
              @clicked="validateCode"
            >
              {{ confirmBtnLabel }}
            </button-submit>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { CodeInput } from 'bedrock-vue-components';

export default {
  components: {
    CodeInput,
  },

  data() {
    return {
      errorMessage: '',
      successMessage: '',
      confirmCode: '',
      confirmCodeInvalid: true,
      submitting: false,

      form: {
        isSubmitting: false,
      },

      apiErrors: {
        form: '',
      },
    };
  },

  props: {
    confirmBtnLabel: {
      type: String,
      default: 'Confirm',
    },
    confirmCodeUrl: {
      type: String,
      required: true,
    },
    cancelBtnLabel: {
      type: String,
      default: 'Cancel',
    },
  },

  methods: {
    onCodeChange() {
      this.confirmCode = '';
    },

    onCodeComplete(value) {
      this.confirmCodeInvalid = false;
      this.confirmCode = value;
    },

    onCodeEnter() {
      this.validateCode();
    },

    cancelAuthenticator() {
      this.$emit('authenticator-form-cancel');
    },

    validateCode() {
      this.submitting = true;
      this.resetErrorState();

      const formData = new FormData();
      formData.set('verify-code', this.confirmCode);
      this.$http
        .post(this.confirmCodeUrl, formData)
        .then(success => {
          if (!success.data.success) {
            this.errorMessage = success.data.message;
            this.clearInput();
          } else {
            this.$emit('authenticator-enabled-successfully');
          }
        })
        .catch(({ response }) => {
          this.errorMessage = response.data.message;
          this.clearInput();
        })
        .finally(() => {
          this.submitting = false;
        });
    },

    clearInput() {
      this.$refs.codeInputComponent.clear();
    },

    resetErrorState() {
      this.apiErrors = {
        form: null,
      };
    },
  },
};
</script>
