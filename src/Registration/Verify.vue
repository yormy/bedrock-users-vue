<template>
  <div>
    <div v-if="xid === '0' || xid === null || xid === ''">
      <h1>{{ $t('confirm_actions.action_already_confirmed.title') }}</h1>
    </div>

    <div v-else>
      <input name="'firstname'" v-model="form.honeypot" type="hidden" />
      <ValidationObserver ref="form">
        <div class="">
          <div v-if="verifyType === 'TOKEN'">
            <ValidationProvider v-slot="{ errors }" rules="required|min:10" name="confirmToken">
              <v-text-field
                autocomplete="confirmToken"
                v-model="confirmToken"
                :label="$t('misc.confirmtoken.field.label')"
                :hint="''"
                :type="'text'"
                :error-messages="errors"
                :color="!apiErrors.confirmToken ? 'success' : ''"
              >
              </v-text-field>
            </ValidationProvider>
          </div>

          <div v-if="verifyType === 'CODE'">
            <div v-if="needsUsername">
              <p>
                {{
                  $t('confirm_actions.confirm.fill_in_loginname', { method: method.toLowerCase() })
                }}
              </p>
            </div>
            <div v-else>
              <p>
                {{
                  $t('confirm_actions.confirm.fill_in_the_method_code', {
                    method: method.toLowerCase(),
                  })
                }}
              </p>
            </div>

            <div class="mb-3">
              <CodeInput
                :loading="form.isSubmittingSubmit"
                :invalid-input="confirmTokenInvalid"
                ref="codeInputComponent"
                class="input"
                type="string"
                :fields="6"
                @change="onCodeChange"
                @complete="onCodeComplete"
                @enter="onCodeEnter"
              />
              <div v-if="apiErrors.confirmCode" class="alert alert-danger">
                {{ apiErrors.confirmCode }}
              </div>
            </div>

            <div v-if="needsUsername">
              <ValidationProvider v-slot="{ errors }" rules="required|min:5|email" name="loginname">
                <v-text-field
                  autocomplete="loginname"
                  v-model="loginname"
                  :label="$t('misc.login.loginname.label')"
                  :hint="$t('misc.login.loginname.hint')"
                  :error-messages="errors"
                  :color="!apiErrors.loginname ? 'success' : ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>
          </div>
        </div>

        <div v-if="apiErrors.form">
          <div class="alert alert-danger" style="overflow-wrap: break-word">
            <strong>{{ apiErrors.form }}</strong>
          </div>
        </div>

        <div v-if="apiSuccess">
          <div class="alert alert-success" style="overflow-wrap: break-word">
            <strong>{{ apiSuccess }}</strong>
          </div>
        </div>

        <div class="d-flex justify-content-between">
          <div>
            <button-submit
              v-if="resendPossible"
              btn-class="btn btn-secondary"
              :is-loading="form.isSubmittingResend"
              @clicked="resendAction"
            >
              {{ $t('misc.confirmtoken.resend') }}
            </button-submit>
          </div>

          <div>
            <button-submit :is-loading="form.isSubmittingSubmit" @clicked="confirmAction">
              {{ $t('misc.confirmtoken.confirm') }}
            </button-submit>
          </div>
        </div>
      </ValidationObserver>
    </div>
  </div>
</template>

<script>
import CodeInput from './CodeInput.vue';
import { mergeErrors } from 'bedrock-vue-components';

export default {
  components: {
    CodeInput,
  },

  props: {
    loginnameProp: String,
    xid: String,

    resendPossible: {
      type: Boolean,
      required: false,
      default: true,
    },

    verifyType: {
      type: String,
      default: 'TOKEN',
    },

    method: {
      type: String,
      default: '',
    },

    email: {
      type: String,
      default: '',
    },

    verifyTokenErrorProp: String,

    verifyActionUrl: String,
    verifyResendActionUrl: String,
  },

  data() {
    return {
      confirmToken: null,
      confirmCode: null,

      form: {
        isSubmittingSubmit: false,
        isSubmittingResend: false,
      },

      loginname: null,
      loginnameErrors: null,

      apiErrors: {
        loginname: null,
        confirmCode: null,
        confirmToken: null,
        form: null,
      },

      apiSuccess: null,

      encryption: null,

      confirmTokenInvalid: false,
    };
  },

  computed: {
    needsUsername() {
      return this.verifyType === 'CODE' && !this.loginnameProp && !this.xid;
    },
  },

  mounted() {
    if (this.loginnameProp) {
      this.loginname = this.loginnameProp;
    }

    this.apiErrors.form = this.verifyTokenErrorProp;
  },

  methods: {
    isFormValid() {
      this.$refs.form.validate();
      return !this.$refs.form.flags.invalid;
    },

    onCodeChange() {
      this.confirmCode = null;
    },

    onCodeComplete(value) {
      this.confirmTokenInvalid = false;
      this.confirmCode = value;
    },

    onCodeEnter() {
      this.confirmAction();
    },

    confirmAction() {
      if (!this.isFormValid() || (!this.confirmToken && !this.confirmCode)) {
        this.confirmTokenInvalid = true;
        return;
      }
      this.form.isSubmittingSubmit = true;

      const data = {
        confirmToken: this.confirmToken,
        loginname: this.loginname,
        confirmCode: this.confirmCode,
        xid: this.xid,
        firstname: this.form.honeypot,
      };

      this.$http
        .post(this.verifyActionUrl, data)
        .then(response => {
          if (response.data.success) {
            this.isSubmitting = false;
            this.resetErrorState();
            this.clearInput();
            this.$emit('verified', response.data);
          }
          this.form.isSubmittingSubmit = false;
        })
        .catch(error => {
          this.resetErrorState();
          this.apiErrors = mergeErrors(error);
          this.confirmCode = false;
          this.form.isSubmittingSubmit = false;
          this.clearInput();
        });
    },

    resendAction() {
      if (!this.isFormValid()) {
        return;
      }

      this.form.isSubmittingResend = true;

      const data = {
        loginname: this.loginname,
        xid: this.xid,
        method: this.method,
        email: this.email,
      };

      this.$http
        .post(this.verifyResendActionUrl, data)
        .then(response => {
          if (response.data.success) {
            this.resetErrorState();
            this.apiSuccess = response.data.message;
          }
          this.form.isSubmittingResend = false;
        })
        .catch(error => {
          this.resetErrorState();
          this.apiErrors.form = error.response.data.message;
          this.form.isSubmittingResend = false;
          this.clearInput();
        });
    },
    clearInput() {
      if (this.verifyType === 'CODE') {
        this.$refs.codeInputComponent.clear();
      }
    },

    resetErrorState() {
      this.apiErrors = {
        loginname: null,
        confirmCode: null,
        confirmToken: null,
        form: null,
      };
      this.apiSuccess = '';
      this.apiErrors.form = '';

      this.form.isSubmittingSubmit = false;
      this.form.isSubmittingResend = false;
    },
  },
};
</script>
