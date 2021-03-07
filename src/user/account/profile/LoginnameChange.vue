<template>
  <div>
    <div class="container">
      <div v-if="!confirmableAction.xid" class="card card-table">
        <loading-overlay :show="form.isSubmitting"></loading-overlay>

        <div class="card-header">
          <span class="fal fa-fw fa-chart-network"></span>
          <p>{{ $t('profile.change.loginname.title') }}</p>
        </div>

        <div class="card-body">
          <p>{{ $t('profile.change.credentials.description_01') }}</p>
          <p>{{ $t('profile.change.credentials.description_02') }}</p>

          <div v-if="tokenErrorMessage">
            <div class="alert alert-danger" style="overflow-wrap: break-word">
              <strong>{{ tokenErrorMessage }}</strong>
            </div>
          </div>

          <ValidationObserver ref="form">
            <div>
              <ValidationProvider v-slot="{ errors }" rules="required|email|min:5" name="email">
                <v-text-field
                  autocomplete="email"
                  v-model="login.loginname"
                  :label="$t('profile.change.loginname.new.label')"
                  :hint="$t('profile.change.loginname.new.hint')"
                  :type="'email'"
                  :error-messages="apiErrors.loginname ? apiErrors.loginname : errors"
                  :color="!apiErrors.loginname && login.loginname ? 'success' : ''"
                  @keydown="apiErrors.loginname = ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>

              <ValidationProvider
                v-slot="{ errors }"
                rules="required|strong-password"
                name="password"
              >
                <v-text-field
                  autocomplete="password"
                  v-model="login.password"
                  :label="$t('profile.change.current_password.label')"
                  :hint="$t('bedrock-users.login.password.hint')"
                  :append-icon="passwordIcon"
                  @click:append="() => (passwordExpose = !passwordExpose)"
                  :type="passwordExpose ? 'text' : 'password'"
                  :error-messages="apiErrors.password ? apiErrors.password : errors"
                  :color="!apiErrors.password ? 'success' : ''"
                  @keydown="apiErrors.password = ''"
                  counter
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>

            <div v-show="authenticatorEnabled">
              {{ $t('bedrock-users.misc.login.authenticator_code.label') }}
              <CodeInput
                :loading="formConfirmCode.isSubmitting"
                :invalid-input="confirmCodeInvalid"
                ref="codeInputComponent"
                class="input"
                type="number"
                :fields="6"
                @change="onCodeChange"
                @complete="onCodeComplete"
                @enter="onCodeEnter"
              />
            </div>

            <div v-if="apiErrors.form">
              <div class="alert alert-danger" style="overflow-wrap: break-word">
                <strong>{{ apiErrors.form }}</strong>
              </div>
            </div>

            <div v-if="successMessage">
              <div class="alert alert-success" style="overflow-wrap: break-word">
                <strong>{{ successMessage }}</strong>
              </div>
            </div>

            <button-submit :is-loading="form.isSubmitting" @clicked="submitChangeRequest">
              {{ $t('bedrock-users.misc.request_change') }}
            </button-submit>
          </ValidationObserver>
        </div>
      </div>

      <div v-else class="card">
        <div class="card-header"></div>
        <div class="card-body">
          <confirm-action
            :confirm-title="confirmableAction.title"
            :confirm-description="confirmableAction.description"
            :method="confirmableAction.method"
            verify-type="CODE"
            :verify-action-url="route('api.v1.member.registration.verify.verify')"
            :verify-resend-action-url="route('api.v1.member.registration.resend')"
            :xid="confirmableAction.xid"
          >
          </confirm-action>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import APICODES from '@consts/apiCodes';
import { CodeInput } from 'bedrock-vue-components';
import { mergeErrors } from 'bedrock-vue-components';

export default {
  components: {
    CodeInput,
  },

  props: {
    changeActionUrl: String,
    tokenErrorMessage: String,

    authenticatorEnabled: {
      type: Boolean,
      required: true,
    },
  },

  data() {
    return {
      confirmCode: '',
      confirmCodeInvalid: false,

      formConfirmCode: {
        isSubmitting: false,
      },

      form: {
        isSubmitting: false,
      },

      expiredMessage: '',

      confirmableAction: {
        xid: '',
        method: '',
        title: '',
        description: '',
      },

      login: {
        loginname: 'new@test.com',
        password: 'Welkom1!',
        // honeypot: '',
        // tokenIpWhitelisting: '',
        // trustBrowser: false,
      },

      passwordExpose: false,

      apiErrors: {},
      successMessage: '',
    };
  },

  computed: {
    passwordIcon() {
      return this.passwordExpose ? 'mdi-eye-off' : 'mdi-eye';
    },
  },

  mounted() {},

  methods: {
    onCodeChange() {
      this.confirmCode = null;
    },

    onCodeComplete(value) {
      this.confirmCodeInvalid = false;
      this.confirmCode = value;
    },

    onCodeEnter() {
      this.submitChangeRequest();
    },

    isFormValid() {
      if (this.authenticatorEnabled && (!this.confirmCode || this.confirmCodeInvalid)) {
        return false;
      }

      this.$refs.form.validate();
      return !this.$refs.form.flags.invalid;
    },

    submitChangeRequest() {
      if (!this.isFormValid()) {
        return;
      }

      this.form.isSubmitting = true;

      const data = {
        loginname: this.login.loginname,
        password: this.login.password,
        authenticatorCode: this.confirmCode,
      };

      this.clearResponses();
      this.$http
        .post(this.changeActionUrl, data)
        .then((response) => {
          if (response.data.success) {
            this.successMessage = response.data.message;
            this.confirmableAction.xid = response.data.data.xid;
            this.confirmableAction.method = response.data.data.method;
            this.confirmableAction.title = response.data.data.title;
            this.confirmableAction.description = response.data.data.description;
            // put on end of stack
            window.setTimeout(() => {
              //              window.location.reload();
            }, 0);
          }
        })
        .catch((error) => {
          this.clearInput();
          this.apiErrors = mergeErrors(error);
          if (error.response.data.data) {
            if (error.response.data.data.code === API_RESPONSE.AUTH_AUTHENTICATOR_MISSING) {
              this.resetLoadingState();
              return;
            }
            if (error.response.data.data.code === API_RESPONSE.AUTH_AUTHENTICATOR_INVALID) {
              this.resetLoadingState();
              return;
            }
          }
          this.resetLoadingState();
        })
        .finally(() => {
          this.resetLoadingState();
        });
    },

    clearResponses() {
      this.apiErrors = {};
      this.successMessage = '';
    },

    clearInput() {
      if (this.$refs.codeInputComponent !== undefined) {
        this.$refs.codeInputComponent.clear();
      }
    },

    resetLoadingState() {
      this.form.isSubmitting = false;
    },
  },
};
</script>
