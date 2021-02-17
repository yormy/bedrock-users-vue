<template>
  <div>
    <div v-if="xid === '0' || xid === null || xid === ''">
      <h1>{{ $t('confirm_actions.action_already_confirmed.title') }}</h1>
    </div>

    <div v-else>
      <input name="'firstname'" v-model="form.honeypot" type="hidden" />

      <ValidationObserver ref="form">
        <div class="container">
          <ValidationProvider v-slot="{ errors }" rules="required|strong-password" name="password">
            <v-text-field
              autocomplete="password"
              v-model="form.password"
              :label="$t('misc.login.password.label')"
              :hint="$t('misc.login.password.hint')"
              :append-icon="passwordExpose ? 'fal fa-eye' : 'fal fa-eye-slash'"
              @click:append="() => (passwordExpose = !passwordExpose)"
              :type="passwordExpose ? 'text' : 'password'"
              :error-messages="apiErrors.password ? apiErrors.password : errors"
              :color="!apiErrors.password && form.password ? 'success' : ''"
              @keydown="apiErrors.password = ''"
              counter
              outlined
            >
            </v-text-field>
          </ValidationProvider>
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

        <!--        <button-->
        <!--          type="submit"-->
        <!--          class="btn btn-success"-->
        <!--          :disabled="isSubmitting"-->
        <!--          @click.prevent="confirmAction"-->
        <!--        >-->
        <!--          Finish-->
        <!--        </button>-->

        <div class="d-flex justify-content-between">
          <button-submit :is-loading="form.isSubmittingSubmit" @clicked="confirmAction">
            {{ $t('misc.confirmtoken.confirm') }}
          </button-submit>
        </div>
      </ValidationObserver>
    </div>
  </div>
</template>

<script>
import { mergeErrors } from 'bedrock-vue-components';

export default {
  props: {
    xid: String,

    verifyActionUrl: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      passwordExpose: false,

      form: {
        isSubmittingSubmit: false,
        password: '',
        honeypot: '',
      },

      apiErrors: {
        password: null,
      },

      apiSuccess: null,
    };
  },

  methods: {
    isFormValid() {
      this.$refs.form.validate();
      return !this.$refs.form.flags.invalid;
    },

    confirmAction() {
      if (!this.isFormValid()) {
        return;
      }
      this.form.isSubmittingSubmit = true;

      const data = {
        newPassword: this.form.password,
        firstname: this.form.honeypot,
        xid: this.xid,
      };

      this.$http
        .post(this.verifyActionUrl, data)
        .then(response => {
          if (response.data.success) {
            this.isSubmitting = false;
            this.resetErrorState();
            this.form.password = '';
            this.$emit('verified', response.data);
          }
          this.form.isSubmittingSubmit = false;
        })
        .catch(error => {
          this.resetErrorState();
          this.apiErrors = mergeErrors(error);
          this.form.isSubmittingSubmit = false;
          this.form.password = '';
        });
    },

    resetErrorState() {
      this.apiErrors = {
        password: null,
      };
      this.apiSuccess = '';
      this.apiErrors.form = '';

      this.form.isSubmittingSubmit = false;
    },
  },
};
</script>
