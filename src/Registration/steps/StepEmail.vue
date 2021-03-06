<template>
  <div class="container">
    <input type="hidden" name="firstname" v-model="registration.honeypot" />

    <ValidationObserver ref="form">
      <email-input
        ref="emailInput"
        :label="$t('bedrock-users.registration.input.email.label')"
        :hint="backendhint"
        :email-address.sync="registration.email"
        :api-errors="apiErrors"
      ></email-input>

      <ValidationProvider v-slot="{ errors }" rules="required|strong-password" name="password">
        <v-text-field
          autocomplete="password"
          v-model="registration.password"
          :label="$t('bedrock-users.registration.input.password.label')"
          :hint="$t('bedrock-users.registration.input.password.hint')"

          :append-icon="passwordExpose ? 'fal fa-eye' : 'fal fa-eye-slash'"
          @click:append="() => (passwordExpose = !passwordExpose)"
          :type="passwordExpose ? 'text' : 'password'"
          :error-messages="apiErrors.passwordNew ? apiErrors.passwordNew : errors"
          :color="!apiErrors.passwordNew && registration.password ? 'success' : ''"
          @keydown="apiErrors.passwordNew = ''"
          counter
          outlined
        >
        </v-text-field>
      </ValidationProvider>

      <ValidationProvider
        v-slot="{ errors }"
        rules="required|confirmed:password"
        name="passwordConfirm"
      >
        <v-text-field
          v-model="registration.passwordConfirm"
          :label="$t('bedrock-users.registration.input.password_confirm.label')"
          :hint="$t('bedrock-users.registration.input.password_confirm.hint')"
          :append-icon="passwordConfirmExpose ? 'fal fa-eye' : 'fal fa-eye-slash'"
          @click:append="() => (passwordConfirmExpose = !passwordConfirmExpose)"
          :type="passwordConfirmExpose ? 'text' : 'password'"
          :error-messages="apiErrors.password ? apiErrors.password : errors"
          :color="!apiErrors.password && registration.passwordConfirm ? 'success' : ''"
          outlined
        ></v-text-field>
      </ValidationProvider>
    </ValidationObserver>

    <ValidationProvider v-slot="{ errors }" rules="agreed" name="termsGeneral" ref="termsGeneral">
      <div>
        {{ $t('bedrock-users.registration.terms.general') }}
        <a :href="route('guest.terms.general')" target="_blank"><span class="fal fa-external-link">&nbsp;</span></a>
        <v-checkbox
          v-model="registration.termsGeneral"
          class="mt-0"
          :label="$t('bedrock-users.registration.input.i_agree_to_above.label')"
          :error-messages="apiErrors.termsGeneral ? apiErrors.termsGeneral : errors"
        >
        </v-checkbox>
      </div>
    </ValidationProvider>

    <ValidationProvider name="termsMarketing" rules="agreed" ref="termsMarketing">
      {{ $t('bedrock-users.registration.terms.marketing') }}
      <a :href="route('guest.terms.marketing')"  target="_blank"><span class="fal fa-external-link">&nbsp;</span></a>
      <div>
        <v-checkbox
          v-model="registration.termsMarketing"
          class="mt-0"
          :label="$t('bedrock-users.registration.input.i_agree_to_above.label')"
          :error-messages="marketing.errorMessage"
        >
          <template v-slot:label>
            <div :class="marketing.errorMessage !== '' ? 'red--text' : ''">
              {{ $t('bedrock-users.registration.input.i_agree_to_above.label') }}
            </div>
          </template>
        </v-checkbox>
      </div>
    </ValidationProvider>

    <div v-if="apiErrors.form">
      <div class="alert alert-danger" style="overflow-wrap: break-word">
        <strong>{{ apiErrors.form }}</strong>
      </div>
    </div>

    <div v-if="emailWarning.form">
      <div class="alert alert-warning" style="overflow-wrap: break-word">
        <strong>{{ emailWarning.form }}</strong>
      </div>
    </div>

    <div class="d-flex justify-space-between mt-3">
      <div>
        <a :href="route('user.login')">
          <button id="btnShowLogin" @click="forgot = false">
            <span class="fal fa-unlock mr-1"></span>
            {{ $t('bedrock-users.login.button') }}
          </button>
        </a>
      </div>
      <div>
        <button-submit :is-loading="form.isSubmitting" @clicked="registerAction">
          {{ $t('bedrock-users.registration.button.register_now') }}
        </button-submit>
      </div>
    </div>



  </div>
</template>

<script>
import { EmailInput, mergeErrors } from 'bedrock-vue-components';

export default {
  components: {
    EmailInput,
  },

  props: {
    registrationActionUrl: String,
  },

  data() {
    return {
      form: {
        isSubmitting: false,
      },

      lang: {
        error: {
          marketing: this.$t('bedrock-users.registration.input.i_agree_to_above.warning_marketing'),
        },
      },

      isRetry: false,

      registration: {
        email: 'test@gmail.com',
        password: 'Welkom1!',
        passwordConfirm: 'Welkom1!',
        termsGeneral: true,
        termsMarketing: true,
        honeypot: '',
      },

      passwordExpose: false,
      passwordConfirmExpose: false,

      marketing: {
        errorMessage: '',
        isRetry: false,
      },

      apiErrors: {},

      emailWarning: {},

      backendhint: '',

      skipWarnings: false,
    };
  },

  methods: {
    isFormValid() {
      let formValid = true;

      // run validateions
      this.$refs.form.validate();
      this.$refs.termsGeneral.validate();
      if (!this.marketing.isRetry) {
        this.$refs.termsMarketing.validate();
      }

      // prevent submit because main form is invalid
      const mainInvalid = this.$refs.form.flags.invalid;
      if (mainInvalid) {
        formValid = false;
      }

      // prevent because terms are not agreed
      if (!this.registration.termsGeneral) {
        formValid = false;
      }

      // user can skip this, but not on the first click
      if (!this.registration.termsMarketing && !this.marketing.isRetry) {
        this.marketing.errorMessage = this.lang.error.marketing;
        formValid = false;
      }

      this.marketing.isRetry = true;
      return formValid;
    },

    registerAction() {
      if (!this.isFormValid()) {
        return;
      }

      this.form.isSubmitting = true;
      this.backendhint = '';

      const data = {
        email: this.registration.email,
        passwordNew: this.registration.password,
        termsGeneralAgreed: this.registration.termsGeneral,
        termsMarketingAgreed: this.registration.termsMarketing,
        firstname: this.registration.honeypot,
        skipWarnings: this.skipWarnings,
      };

      this.$http
        .post(this.registrationActionUrl, data)
        .then((response) => {
          if (response.data.success) {
            this.$emit('finished', response.data.data);
          }
        })
        .catch((error) => {
          this.apiErrors = mergeErrors(error);

          if (error.response.data.data && error.response.data.data.warnings) {
            this.emailWarning = error.response.data.data.warnings;
            this.backendhint = this.emailWarning.email.message;
            this.skipWarnings = true;
            this.$refs.emailInput.$el.focus();
          }
        })
        .finally(() => {
          this.form.isSubmitting = false;
        });
    },
  },
};
</script>
