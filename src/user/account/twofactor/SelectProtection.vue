<template>
  <div>

    <div class="card">
      <div class="card-header">
        <v-col cols="6">
          <span class="fal fa-lock"></span> {{ $t('bedrock-users.profile.security.select.title') }}
        </v-col>
      </div>

      <div class="card-body">
<!--    <div class="vx-col w-full lg:w-1/2 xl:w-1/2">-->
<!--      <div class="vx-card overflow-hidden mb-base">-->

        <div class="d-flex justify-content-between my-3">
          <div>
            <h2 class="mb-1 font-bold">{{ $t('bedrock-users.profile.security.select.whitelisting.title') }}</h2>
            <span>{{ $t('bedrock-users.profile.security.select.whitelisting.description') }}</span>
            <div id="'ipwhitelisting-activation-section" v-if="showWhitelistingEnable">
              <div class="mt-4">
                <google-authenticator-disable-form
                  :confirm-url="authenticatorDisableUrl"
                  @whitelisting-form-cancel="hideWitelistingEnableForm"
                  @whitelisting-enabled-successfully="whitelistingEnabledSuccess"
                ></google-authenticator-disable-form>
              </div>
            </div>
          </div>

          <div class="">
            <div @click="enableWhitelisting">
              <v-switch
                v-model="whitelistingEnabled"
                color="success"
                :readonly="true"
                hide-details
              ></v-switch>
            </div>
          </div>
        </div>

        <div class="d-flex justify-content-between my-3">
          <div>
            <h2 class="mb-1 font-bold">{{ $t('bedrock-users.profile.security.select.authenticator.title') }}</h2>
            <span>{{ $t('bedrock-users.profile.security.select.authenticator.description') }}</span>
            {{ showAuthenticatorEnable }}
            <div id="authenticator-activation-section" v-if="showAuthenticatorEnable">
              <!--            @include('security::google2fa.enable')-->

              <div class="my-3">
                <strong>{{ $t('bedrock-users.profile.security.authenticator.enable.step_1') }}</strong>
                <div>
                  <a
                    href="'https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2'"
                    target="_blank"
                  >
                    <img
                      class="h-12"
                      src="/modules/security/images/store-android.jpg"
                      alt="google play"
                      width="200"
                    />
                  </a>

                  <a
                    href="'https://apps.apple.com/nl/app/google-authenticator/id388497605'"
                    target="_blank"
                  >
                    <img
                      class="h-12"
                      src="/modules/security/images/store-apple.jpg"
                      alt="itunes"
                      width="200"
                    />
                  </a>
                </div>
                <br />
                <strong>{{ $t('bedrock-users.profile.security.authenticator.enable.step_2') }}</strong>
              </div>
              <div>
                <google-authenticator-qr
                  :qr-url="authenticatorQrUrl"
                  @loaded="qrLoaded = true"
                ></google-authenticator-qr>
              </div>

              <strong>{{ $t('bedrock-users.profile.security.authenticator.enable.step_3') }}</strong>
              <google-authenticator-form
                v-if="qrLoaded"
                :confirm-code-url="authenticatorEnableUrl"
                @authenticator-form-cancel="hideAuthenticationEnableForm"
                @authenticator-enabled-successfully="authenticatorEnabledSuccessfully"
              >
              </google-authenticator-form>
            </div>
          </div>
          <div class="">
            <div @click="enableAuthenticator">
              <v-switch
                v-model="authenticatorEnabled"
                color="success"
                hide-details
                :readonly="true"
              ></v-switch>
            </div>
          </div>
        </div>
<!--      </div>-->
<!--    </div>-->
    </div>

    </div>
  </div>
</template>

<script>
import GoogleAuthenticatorQr from './authenticator/GoogleAuthenticatorQr.vue';
import GoogleAuthenticatorForm from './authenticator/GoogleAuthenticatorForm.vue';
import GoogleAuthenticatorDisableForm from './authenticator/GoogleAuthenticatorDisableForm.vue';

export default {
  components: {
    GoogleAuthenticatorQr,
    GoogleAuthenticatorForm,
    GoogleAuthenticatorDisableForm,
  },

  props: {
    authenticatorEnableUrl: {
      type: String,
      required: true,
    },

    authenticatorDisableUrl: {
      type: String,
      required: true,
    },

    authenticatorQrUrl: {
      type: String,
      required: true,
    },

    initWhitelistingEnabled: {
      type: Boolean,
      required: false,
    },

    initAuthenticatorEnabled: {
      type: Boolean,
      required: false,
    },
  },

  data() {
    return {
      errorMessage: '',
      qrLoaded: false,
      successMessage: '',
      confirmCode: '',
      showWhitelistingEnable: false,
      showAuthenticatorEnable: false,
      whitelistingEnabled: false,
      authenticatorEnabled: false,

      btnLoading: false,
    };
  },

  created() {
    if (this.initAuthenticatorEnabled) {
      this.setAuthenticatorActive();
    } else {
      this.setWhitelistingActive();
    }
  },

  watch: {
    initWhitelistingEnabled() {
      this.whitelistingEnabled = this.initWhitelistingEnabled;
    },
    initAuthenticatorEnabled() {
      this.authenticatorEnabled = this.initAuthenticatorEnabled;
    },
  },

  methods: {
    hideWitelistingEnableForm() {
      this.showWhitelistingEnable = false;
    },

    hideAuthenticationEnableForm() {
      this.showAuthenticatorEnable = false;
    },

    authenticatorEnabledSuccessfully() {
      this.hideAuthenticationEnableForm();
      this.setAuthenticatorActive();
      window.location.reload();
    },

    whitelistingEnabledSuccess() {
      this.setWhitelistingActive();
      window.location.reload();
    },

    enableAuthenticator() {
      if (!this.authenticatorEnabled) {
        this.showAuthenticatorEnable = true;
      }
    },

    enableWhitelisting(event) {
      if (!this.whitelistingEnabled) {
        this.showWhitelistingEnable = true;
      }
      event.preventDefault();
    },

    setAuthenticatorActive() {
      this.whitelistingEnabled = false;
      this.authenticatorEnabled = true;
    },

    setWhitelistingActive() {
      this.whitelistingEnabled = true;
      this.authenticatorEnabled = false;
    },
  },
};
</script>

<style scoped>
.offWarning {
  background-color: red;
}
</style>
