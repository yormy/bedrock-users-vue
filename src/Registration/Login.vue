<template>
  <div>
    <v-dialog v-model="sessionExpiredMessage" width="500" persistent>
      <v-card>
        <v-card-title class="headline">Session expired</v-card-title>

        <v-card-text>
          <p>Doordat er geen activiteit is gedetecteerd is je sessie verlopen</p>
          <button @click="sessionExpiredMessage = ''" class="btn btn-success">Login Again</button>
        </v-card-text>
      </v-card>
    </v-dialog>

    <v-dialog v-model="reloginChangedMessage" width="500" persistent>
      <v-card>
        <v-card-title class="headline">Login Credentials Changed</v-card-title>

        <v-card-text>
          <p>
            Login information has changed, please re-login with your new credentials to continue
          </p>
          <button @click="reloginChangedMessage = ''" class="btn btn-success">Login Again</button>
        </v-card-text>
      </v-card>
    </v-dialog>

    <div class="container">
      <div v-if="showDecrypting">
        <br />
        <div class="">
          <h1>Processing...</h1>
          <appearing-bullets :messages="messages" :reset="showDecrypting"></appearing-bullets>
        </div>
      </div>

      <div v-else>
        <div v-if="login.tokenIpWhitelisting">
          <h1>LOGIN and whitelist your ip</h1>
        </div>
        <div v-else>
          <h1>LOGIN VUE</h1>
        </div>

        <div v-if="tokenErrorMessage">
          <div class="alert alert-danger" style="overflow-wrap: break-word">
            <strong>{{ tokenErrorMessage }}</strong>
          </div>
        </div>

        <input name="'firstname'" v-model="login.honeypot" type="hidden" />

        <ValidationObserver ref="form">
          <div v-if="showProvideCredentials">
            <ValidationProvider v-slot="{ errors }" rules="required|min:5" name="loginname">
              <v-text-field
                autocomplete="email"
                v-model="login.loginname"
                :label="$t('misc.login.loginname.label')"
                :hint="$t('misc.login.loginname.hint')"
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
                :label="$t('misc.login.password.label')"
                :hint="$t('misc.login.password.hint')"
                :append-icon="passwordIcon"
                @click:append="() => (passwordExpose = !passwordExpose)"
                :type="passwordExpose ? 'text' : 'password'"
                :error-messages="apiErrors.password ? apiErrors.password : errors"
                :color="!apiErrors.password && login.password ? 'success' : ''"
                @keydown="apiErrors.password = ''"
                counter
                outlined
              >
              </v-text-field>
            </ValidationProvider>
          </div>

          <div v-show="showProvideAuthenticatorCode">
            Type your google authenticator code
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

          <div v-if="login.tokenIpWhitelisting">
            <div v-if="tokenErrorMessage">
              <ValidationProvider
                v-slot="{ errors }"
                rules="required|alpha_num|min:16"
                name="tokenIpWhitelisting"
              >
                <v-text-field
                  v-model="login.tokenIpWhitelisting"
                  :label="$t('misc.login.authenticator_code.label')"
                  :error-messages="
                    apiErrors.tokenIpWhitelisting ? apiErrors.tokenIpWhitelisting : errors
                  "
                  :color="
                    !apiErrors.tokenIpWhitelisting && login.tokenIpWhitelisting ? 'success' : ''
                  "
                  @keydown="apiErrors.tokenIpWhitelisting = ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>

            <div class="form-group row mb-3">
              <div class="col-sm-12">
                <v-switch v-model="login.trustBrowser" color="success" hide-details></v-switch>
                {{ $t('misc.login.security.trust_this_browser.title') }}
              </div>
              <div class="col-sm-12">
                <small class="form-text text-muted">
                  {{ $t('misc.login.security.trust_this_browser.description') }}
                </small>
              </div>
            </div>
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

          <button-submit id="btnLogin" :is-loading="form.isSubmitting" @clicked="loginAction">
            {{ $t('misc.login.button') }}
          </button-submit>
        </ValidationObserver>
      </div>
    </div>
  </div>
</template>

<script>
import {
  AppearingBullets,
  removeTokensUser,
  removeTokensAdmin,
  storeTokens,
  mergeErrors,
} from 'bedrock-vue-components';

// todo npm
import API_RESPONSE from '@components/helpers/apiCodes';
import CodeInput from './CodeInput.vue';

export default {
  components: {
    CodeInput,
    AppearingBullets,
  },

  props: {
    loginActionUrl: String,
    tokenIpWhitelisting: String,
    tokenErrorMessage: String,
    loginas: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      showProvideCredentials: false,
      showProvideAuthenticatorCode: false,

      confirmCode: '',
      confirmCodeInvalid: false,

      formConfirmCode: {
        isSubmitting: false,
      },

      form: {
        isSubmitting: false,
      },

      sessionExpiredMessage: '',
      reloginChangedMessage: '',

      login: {
        loginname: 'gold@test.com',
        password: 'Welkom1!',
        honeypot: '',
        tokenIpWhitelisting: '',
        trustBrowser: false,
      },

      passwordExpose: false,

      apiErrors: {},
      successMessage: '',
      showDecrypting: false,

      messages: [
        {
          text: 'Validating your credentials',
          loading: false,
          completed: false,
          processingMs: 3000,
        },
        {
          text: 'Decrypting your data',
          loading: false,
          completed: false,
          processingMs: 2000,
        },
        {
          text: 'Increasing your security level',
          loading: false,
          completed: false,
          processingMs: 1000,
        },
      ],
    };
  },

  computed: {
    passwordIcon() {
      return this.passwordExpose ? 'mdi-eye-off' : 'mdi-eye';
    },
  },

  mounted() {
    if (this.loginas === 'ADMIN') {
      removeTokensAdmin();
    } else {
      removeTokensUser();
    }

    this.login.tokenIpWhitelisting = this.tokenIpWhitelisting;
    this.showCredentials();

    const messageQuery = this.getQueryParams('message', window.location.href);
    if (messageQuery === 'session-expired') {
      this.sessionExpiredMessage = 'Session Expired';
    }
    if (messageQuery === 'relogin-changed') {
      this.reloginChangedMessage = 'Session Expired';
    }
  },

  methods: {
    getQueryParams(params, url) {
      const href = url;
      // this expression is to get the query strings
      const reg = new RegExp(`[?&]${params}=([^&#]*)`, 'i');
      const queryString = reg.exec(href);
      return queryString ? queryString[1] : null;
    },

    onCodeChange() {
      this.confirmCode = null;
    },

    onCodeComplete(value) {
      this.confirmCodeInvalid = false;
      this.confirmCode = value;
    },

    onCodeEnter() {
      this.loginAction();
    },

    isFormValid() {
      this.$refs.form.validate();
      return !this.$refs.form.flags.invalid;
    },

    getCookie(name) {
      const value = `; ${document.cookie}`;
      const parts = value.split(`; ${name}=`);
      let cookieValue = '';
      if (parts.length === 2) {
        cookieValue = parts.pop().split(';').shift();
      }

      if (cookieValue !== undefined) {
        return cookieValue;
      }
      return '';
    },

    loginAction() {
      if (!this.isFormValid()) {
        return;
      }

      this.startLoadingState();

      const data = {
        loginname: this.login.loginname,
        password: this.login.password,
        firstname: this.login.honeypot,

        authenticatorCode: this.confirmCode,

        tokenIpWhitelisting: this.login.tokenIpWhitelisting,
        skipRetry: true,
        trustBrowser: this.login.trustBrowser,
        trust_token: `${this.getCookie('trust_token')}`, // needs insecure cookie to read
      };

      this.clearResponses();
      this.$http
        .post(this.loginActionUrl, data)
        .then((response) => {
          if (response.data.success) {
            this.successMessage = response.data.message;
            storeTokens(response.data.data, this.loginas);
            // put on end of stack
            window.setTimeout(() => {
              window.location.reload();
            }, 0);
          }
        })
        .catch((error) => {
          this.clearInput();
          this.apiErrors = mergeErrors(error);
          if (error.response.data.data) {
            if (error.response.data.data.code === API_RESPONSE.AUTH_AUTHENTICATOR_MISSING) {
              if (this.showProvideCredentials) {
                this.apiErrors = {}; // hide error on first try
              }
              this.resetLoadingState();
              this.showAuthenticatorCode();
              return;
            }
            if (error.response.data.data.code === API_RESPONSE.AUTH_AUTHENTICATOR_INVALID) {
              this.resetLoadingState();
              this.showAuthenticatorCode();
              return;
            }
          }
          this.resetLoadingState();
          this.showCredentials();
        })
        .finally(() => {});
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

    startLoadingState() {
      this.form.isSubmitting = true;
      this.showDecrypting = true;
      this.$emit('submitting', true);
    },

    resetLoadingState() {
      this.showDecrypting = false;
      this.form.isSubmitting = false;
      this.$emit('submitting', false);
    },

    showCredentials() {
      this.showProvideCredentials = true;
      this.showProvideAuthenticatorCode = false;
    },

    showAuthenticatorCode() {
      this.showProvideCredentials = false;
      this.showProvideAuthenticatorCode = true;
    },
  },
};
</script>
