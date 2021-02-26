<template>
  <div>
    <div class="container">
      <div v-if="!confirmableAction.xid" class="card card-table">
        <loading-overlay :show="form.isSubmitting"></loading-overlay>

        <div class="card-header">
          <span class="fal fa-fw fa-chart-network"></span>
          <p>{{ $t('profile.change.password.title') }}</p>
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

              <ValidationProvider
                v-slot="{ errors }"
                rules="required|strong-password"
                name="password"
              >
                <v-text-field
                  autocomplete="password"
                  v-model="form.currentPassword"
                  :label="$t('profile.change.current_password.label')"
                  :hint="$t('misc.login.password.hint')"
                  :append-icon="passwordCurrentExpose ? 'fal fa-eye' : 'fal fa-eye-slash'"
                  @click:append="() => (passwordCurrentExpose = !passwordCurrentExpose)"
                  :type="passwordCurrentExpose ? 'text' : 'password'"
                  :error-messages="apiErrors.currentPassword ? apiErrors.currentPassword : errors"
                  :color="!apiErrors.currentPassword ? 'success' : ''"
                  @keydown="apiErrors.currentPassword = ''"
                  counter
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>

            <div v-show="authenticatorEnabled">
              {{ $t('misc.login.authenticator_code.label') }}
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
              {{ $t('misc.request_change') }}
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
            :verify-action-url="route('api.v1.guest.verify')"
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
import { SecureChange } from 'bedrock-vue-components';

export default {
  extends: SecureChange,

  data() {
    return {
      passwordNewExpose: false,
      passwordConfirmExpose: false,
      passwordCurrentExpose: false,

      form: {
        newPassword: 'Welkom2!',
        confirmPassword: 'Welkom2!',
        currentPassword: 'Welkom1!',
      },
    };
  },

  methods: {
    isInputValid() {
      return true;
    },

    getData() {
      const data = {
        newPassword: this.form.newPassword,
        currentPassword: this.form.currentPassword,
        authenticatorCode: this.confirmCode,
      };

      return data;
    },
  },
};
</script>
