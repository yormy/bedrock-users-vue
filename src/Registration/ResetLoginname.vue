<template>
  <div>
    <div class="">
      <div v-if="!confirmableAction.xid">
        <input name="'firstname'" v-model="form.honeypot" type="hidden" />

        <ValidationObserver ref="form">
          <ValidationProvider v-slot="{ errors }" rules="required|min:8" name="loginname">
            <v-text-field
              v-model="form.loginname"
              :label="$t('profile.change.loginname.new.label')"
              :hint="$t('profile.change.loginname.new.hint')"
              append-icon="fal fa-user"
              :error-messages="apiErrors.loginname ? apiErrors.loginname : errors"
              :color="!apiErrors.loginname && form.loginname ? 'success' : ''"
              @keydown="apiErrors.loginname = ''"
              counter
              outlined
            >
            </v-text-field>
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
            @clicked="resetLoginnameAction"
          >
            {{ $t('profile.reset.loginname.button') }}
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
            :verify-loginname-action-url="route('api.v1.guest.verify.loginname')"
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
import { mergeErrors } from 'bedrock-vue-components';
import EmailInput from './EmailInput.vue';

export default {
  components: {
    EmailInput,
  },

  props: {
    resetLoginnameRequestUrl: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      form: {
        loginname: 'testing123',
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

      emailWarnings: 0,
      validateNow: false,
      preventSubmitOnWarning: true,
    };
  },

  created() {
    this.clearResult();
  },

  methods: {
    isFormValid() {
      this.validateNow = true;

      // do not submit on first created warnings, can submit on subsequent warnings
      if (this.emailWarnings === 1 && this.preventSubmitOnWarning) {
        this.preventSubmitOnWarning = false;
        return false;
      }

      this.$refs.form.validate();
      return !this.$refs.form.flags.invalid;
    },

    resetLoginnameAction() {
      if (!this.isFormValid()) {
        return;
      }

      this.clearResult();
      this.setLoadingState();

      const data = {
        loginname: this.form.loginname,
        email: this.form.email,
        firstname: this.form.honeypot,
      };

      this.$http
        .post(this.resetLoginnameRequestUrl, data)
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
          this.skipWarnings = true;
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
