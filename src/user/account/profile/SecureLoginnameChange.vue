<template>
  <div>
    <div class="container">
      <div class="col-6">
        <div v-if="!confirmableAction.xid" class="card card-table">
        <loading-overlay :show="form.isSubmitting"></loading-overlay>

        <div class="card-header">
          <span class="fal fa-fw fa-user"></span>
          {{ $t('bedrock-users.profile.security.change_loginname') }}
        </div>

        <div class="card-body">
          <p>{{ $t('bedrock-users.profile.change.credentials.description_01') }}</p>
          <p>{{ $t('bedrock-users.profile.change.credentials.description_02') }}</p>

          <div v-if="tokenErrorMessage">
            <div class="alert alert-danger" style="overflow-wrap: break-word">
              <strong>{{ tokenErrorMessage }}</strong>
            </div>
          </div>

          <ValidationObserver ref="form">
            <div>
              <ValidationProvider v-slot="{ errors }" rules="required|email|min:5" name="email">
                <v-text-field
                  ref="emailInput"
                  autocomplete="email"
                  v-model="form.loginname"
                  :label="$t('bedrock-users.profile.change.email.new.label')"
                  :hint="$t('bedrock-users.profile.change.email.new.hint')"
                  :type="'email'"
                  :error-messages="apiErrors.loginname ? apiErrors.loginname : errors"
                  :color="!apiErrors.loginname && form.loginname ? 'success' : ''"
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
                  v-model="form.currentPassword"
                  :label="$t('bedrock-users.profile.change.current_password.label')"
                  :append-icon="passwordConfirmExpose ? 'fal fa-eye' : 'fal fa-eye-slash'"
                  @click:append="() => (passwordExpose = !passwordExpose)"
                  :type="passwordExpose ? 'text' : 'password'"
                  :error-messages="apiErrors.currentPassword ? apiErrors.currentPassword : errors"
                  :color="!apiErrors.currentPassword ? 'success' : ''"
                  @keydown="apiErrors.currentPassword = ''"
                  counter
                  outlined
                >
                </v-text-field>
              </ValidationProvider>

              <div v-if="emailWarning.form">
                <div class="alert alert-warning" style="overflow-wrap: break-word">
                  <strong>{{ emailWarning.form }}</strong>
                </div>
              </div>
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

            <div class="float-right">
              <button-submit :is-loading="form.isSubmitting" @clicked="submitChangeRequest">
                {{ $t('bedrock-users.action.request_change') }}
              </button-submit>
            </div>
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
  </div>
</template>

<script>
import { SecureChange } from 'bedrock-vue-components';;

export default {
  extends: SecureChange,

  data() {
    return {
      form: {
        loginname: null,
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
        loginname: this.form.loginname,
        currentPassword: this.form.currentPassword,
        authenticatorCode: this.confirmCode,
        skipWarnings: this.skipWarnings,
      };

      return data;
    },
  },
};
</script>
