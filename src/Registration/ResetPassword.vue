<template>
  <div>
    <div class="">
      <div v-if="!confirmableAction.xid">
        <input name="'firstname'" v-model="form.honeypot" type="hidden" />

        <ValidationObserver ref="form">
          <ValidationProvider
            v-slot="{ errors }"
            rules="required|strong-password"
            name="newPassword"
          >
            <v-text-field
              autocomplete="password"
              v-model="form.newPassword"
              :label="$t('misc.login.password.label')"
              :hint="$t('misc.login.password.hint')"
              :append-icon="passwordNewExpose ? 'fal fa-eye' : 'fal fa-eye-slash'"
              @click:append="() => (passwordNewExpose = !passwordNewExpose)"
              :type="passwordNewExpose ? 'text' : 'password'"
              :error-messages="apiErrors.newPassword ? apiErrors.newPassword : errors"
              :color="!apiErrors.newPassword && form.newPassword ? 'success' : ''"
              @keydown="apiErrors.newPassword = ''"
              counter
              outlined
            >
            </v-text-field>
          </ValidationProvider>

          <ValidationProvider
            v-slot="{ errors }"
            rules="required|confirmed:newPassword"
            name="passwordConfirm"
          >
            <v-text-field
              v-model="form.confirmPassword"
              :label="$t('misc.registration.re-enter-password')"
              :hint="$t('misc.login.password.hint')"
              :append-icon="passwordConfirmExpose ? 'fal fa-eye' : 'fal fa-eye-slash'"
              @click:append="() => (passwordConfirmExpose = !passwordConfirmExpose)"
              :type="passwordConfirmExpose ? 'text' : 'password'"
              :error-messages="apiErrors.confirmPassword ? apiErrors.confirmPassword : errors"
              :color="!apiErrors.confirmPassword && form.passwordConfirm ? 'success' : ''"
              outlined
            ></v-text-field>
          </ValidationProvider>

          <email-input
            id="fieldCurrentEmail"
            :label="$t('profile.current_emailaddress')"
            :email-address.sync="form.email"
            :hint="apiErrors.email"
            :api-errors="apiErrors"
          ></email-input>

          <div v-if="apiErrors.form">
            <div class="alert alert-danger" style="overflow-wrap: break-word">
              <strong>{{ apiErrors.form }}</strong>
            </div>
          </div>

          <button-submit
            id="btnResetLoginname"
            :is-loading="form.isSubmitting"
            @clicked="resetPasswordAction"
          >
            {{ $t('profile.reset.password.button') }}
          </button-submit>
        </ValidationObserver>
      </div>

      <div v-else class="card">
        <div class="card-header"></div>
        <div class="card-body">
          <confirm-action
            :confirm-title="confirmableAction.title"
            :confirm-description="confirmableAction.description"
            :method="confirmableAction.method"
            :email="form.email"
            verify-type="CODE"
            :verify-action-url="route('api.v1.guest.verify')"
            :verify-password-action-url="route('api.v1.guest.verify.password')"
            :verify-resend-action-url="route('api.v1.guest.resend')"
            :xid="confirmableAction.xid"
          >
          </confirm-action>
        </div>
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
    resetPasswordRequestUrl: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      passwordNewExpose: false,
      passwordConfirmExpose: false,

      form: {
        newPassword: 'Welkom2!',
        confirmPassword: 'Welkom2!',
        email: 'gold@test.com',
        honeypot: '',
        isSubmitting: false,
      },

      confirmableAction: {
        xid: '',
        method: '',
        title: '',
        description: '',
      },

      result: {},

      apiErrors: {},

      skipWarnings: false,
    };
  },

  created() {
    this.clearResult();
  },

  methods: {
    isFormValid() {
      this.validateNow = true;

      this.$refs.form.validate();
      return !this.$refs.form.flags.invalid;
    },

    resetPasswordAction() {
      if (!this.isFormValid()) {
        return;
      }

      this.clearResult();
      this.setLoadingState();

      const data = {
        newPassword: this.form.newPassword,
        email: this.form.email,
        firstname: this.form.honeypot,
      };

      this.$http
        .post(this.resetPasswordRequestUrl, data)
        .then((response) => {
          if (response.data.success) {
            this.successMessage = response.data.message;
            this.confirmableAction.xid = response.data.data.confirmableActionXid;
            this.confirmableAction.method = response.data.data.method;
            this.confirmableAction.title = response.data.data.title;
            this.confirmableAction.description = response.data.data.description;
          }
        })
        .catch((error) => {
          this.apiErrors = mergeErrors(error);
        })
        .finally(() => {
          this.clearLoadingState();
        });
    },

    setLoadingState() {
      this.form.isSubmitting = true;
    },

    clearLoadingState() {
      this.form.isSubmitting = false;
    },

    clearResult() {
      this.apiErrors = {};
    },
  },
};
</script>
