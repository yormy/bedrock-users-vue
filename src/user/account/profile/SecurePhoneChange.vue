<template>
  <div>
    <div class="container">
      <div class="col-6">
        <div v-if="!confirmableAction.xid" class="card card-table">
        <loading-overlay :show="form.isSubmitting"></loading-overlay>

        <div class="card-header">
          <span class="fal fa-fw fa-phone"></span>
          {{ $t('bedrock-users.profile.security.phone_change.title') }}
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
              <maz-phone-number-input
                v-model="phoneNumerUnformatted"
                size="lg"
                valid-color="#459B19"
                error-color="#FF0000"
                :required="true"
                @update="phoneUpdated"
                :default-country-code="'NL'"
                :translations="$t('bedrock-core.mazui.phone')"
              />

              <div class="my-3">
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
                  @keydown="apiErrors.currentPassword = ''; phoneUpdated"
                  counter
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
              </div>

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
              <button-submit :is-loading="form.isSubmitting" @clicked="submitChangeRequest" class="">
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
import { SecureChange } from 'bedrock-vue-components';

export default {
  extends: SecureChange,

  data() {
    return {
      phoneNumerUnformatted: '612299899',
      phoneValid: false,

      form: {
        phone: '',
        currentPassword: 'Welkom1!',
      },
    };
  },

  methods: {
    isInputValid() {
      return this.phoneValid;
    },

    phoneUpdated(data) {
      this.phoneValid = data.isValid;

      if (data.isValid) {
        this.form.phone = data.e164;
        this.apiErrors.form = '';
      } else {
        this.apiErrors.form = this.$t('bedrock-core.validations.phone.invalid');
      }
    },

    getData() {
      const data = {
        phone: this.form.phone,
        currentPassword: this.form.currentPassword,
        authenticatorCode: this.confirmCode,
        skipWarnings: this.skipWarnings,
      };

      return data;
    },
  },
};
</script>
