<template>
  <div>
    <div id="authenticator-deactivation-section-form">
      <div>
        <div class="col-12 mt-3">
          <ValidationObserver ref="form">
            <ValidationProvider rules="required|strong-password" name="password">
              <v-text-field
                v-model="currentPassword"
                :label="$t('misc.login.password.label')"
                :append-icon="passwordIcon"
                @click:append="() => (passwordExpose = !passwordExpose)"
                :type="passwordExpose ? 'text' : 'password'"
                @keydown="errorMessage = ''"
                @keyup.enter="disableGoogleAuthenticator"
                counter
                outlined
              >
              </v-text-field>
            </ValidationProvider>
          </ValidationObserver>

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

        <div class="d-flex justify-content-between">
          <button
            id="ipwhitelisting-cancel-button"
            class="vs-component vs-button vs-button-primary vs-button-border"
            @click="hideWhitelistingEnableForm"
          >
            {{ $t('actions.cancel') }}
          </button>

          <button-submit
            id="submit"
            :is-loading="form.isSubmitting"
            :disabled="!currentPassword"
            @clicked="disableGoogleAuthenticator"
          >
            {{ $t('actions.activate') }}
          </button-submit>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import CryptoJS from 'crypto-js';

export default {
  data() {
    return {
      errorMessage: '',
      successMessage: '',
      currentPassword: '',
      passwordExpose: false,

      form: {
        isSubmitting: false,
      },

      apiErrors: {},
    };
  },

  props: {
    confirmUrl: {
      type: String,
      required: true,
    },
  },

  computed: {
    passwordIcon() {
      return this.passwordExpose ? 'mdi-eye-off' : 'mdi-eye';
    },
  },

  methods: {
    hideWhitelistingEnableForm() {
      this.$emit('whitelisting-form-cancel');
    },

    // sha512(message) {
    //   return CryptoJS.SHA512(message).toString();
    // },

    disableGoogleAuthenticator() {
      if (!this.currentPassword) {
        return;
      }

      this.form.isSubmitting = true;
      this.clearErrors();

      const data = {
        // currentPassword: this.sha512(this.currentPassword),
        currentPassword: this.currentPassword,
      };

      this.$http
        .post(this.confirmUrl, data)
        .then(success => {
          if (!success.data.success) {
            this.errorMessage = success.data.message;
            this.clearInput();
          } else {
            this.$emit('whitelisting-enabled-successfully');
            this.hideWhitelistingEnableForm();
          }
        })
        .catch(({ response }) => {
          this.errorMessage = response.data.message;
          this.clearInput();
        })
        .finally(() => {
          this.form.isSubmitting = false;
        });
    },

    clearInput() {
      //  this.$refs.codeInputComponent.clear();
    },
    clearErrors() {
      this.errorMessage = '';
    },
  },
};
</script>
