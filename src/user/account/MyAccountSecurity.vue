<template>
  <div>
    <v-tabs
      v-model="tab"
      background-color="bg-warning"
      class="elevation-2"
      dark
      :grow="true"
      :icons-and-text="true"
    >
      <v-tabs-slider></v-tabs-slider>

      <v-tab :href="`#tab-credentials`">
        {{ $t('bedrock-users.profile.credentials') }}
        <i class="fal fa-user fa-2x"></i>
      </v-tab>

      <v-tab :href="`#tab-protection`">
        {{ $t('bedrock-users.profile.protection')}}
        <i class="fal fa-lock fa-2x"></i>
      </v-tab>

      <v-tab-item :value="'tab-credentials'">
        <div class="my-3">
          <div @click="showSection('phoneChange')" class="btn btn-primary">
            {{ $t('bedrock-users.profile.security.phone_change.title') }}
          </div>
          <div @click="showSection('emailChange')" class="btn btn-primary">
            {{ $t('bedrock-users.profile.security.email_change.title') }}
          </div>
          <div @click="showSection('loginnameChange')" class="btn btn-primary">
            {{ $t('bedrock-users.profile.security.change_loginname') }}
          </div>
          <div @click="showSection('passwordChange')" class="btn btn-primary">
            {{ $t('bedrock-users.profile.security.change_password')  }}
          </div>

          <div v-if="show.phoneChange">
            <secure-phone-change
              :change-action-url="phoneChangeUrl"
              :authenticator-enabled="authenticatorEnabled"
            >
            </secure-phone-change>
          </div>

          <div v-if="show.emailChange">
            <secure-email-change
              :user="user"
              :change-action-url="emailChangeUrl"
              :authenticator-enabled="authenticatorEnabled"
            >
            </secure-email-change>
          </div>

          <div v-if="loginnameChangeUrl">
            <div v-if="show.loginnameChange">
              <secure-loginname-change
                :user="user"
                :change-action-url="loginnameChangeUrl"
                :authenticator-enabled="authenticatorEnabled"
              >
              </secure-loginname-change>
            </div>
          </div>

          <div v-if="loginnameChangeUrl">
            <div v-if="show.passwordChange">
              <secure-password-change
                :user="user"
                :change-action-url="passwordChangeUrl"
                :authenticator-enabled="authenticatorEnabled"
              >
              </secure-password-change>
            </div>
          </div>
      </div>
      </v-tab-item>

      <v-tab-item :value="'tab-protection'">
        <select-protection
          :init-authenticator-enabled="authenticatorEnabled"
          :authenticator-disable-url="authenticatorDisableUrl"
          :authenticator-enable-url="authenticatorEnableUrl"
          :authenticator-qr-url="authenticatorQrUrl"
        >
        </select-protection>

        <h1>{{ $t('bedrock-users.profile.security.whitelisting.title') }}</h1>
        <v-row>
          <v-col>
            <security-whitelisted-ip
              :datatableValues="whitelistedIps"
              :url-whitelisted-ip-delete="urlWhitelistedIpDelete"
            ></security-whitelisted-ip>
          </v-col>
          <v-col>
            <security-whitelisted-browser
              :datatableValues="whitelistedBrowsers"
              :url-whitelisted-browser-delete="urlWhitelistedBrowserDelete"
            ></security-whitelisted-browser>
          </v-col>
        </v-row>
      </v-tab-item>
    </v-tabs>
  </div>
</template>

<script>
import SecurityWhitelistedIp from './profile/SecurityWhitelistedIp.vue';
import SecurityWhitelistedBrowser from './profile/SecurityWhitelistedBrowser.vue';
import SecureEmailChange from './profile/SecureEmailChange.vue';
import SecurePhoneChange from './profile/SecurePhoneChange.vue';
import SecureLoginnameChange from './profile/SecureLoginnameChange.vue';
import SecurePasswordChange from './profile/SecurePasswordChange.vue';
import SelectProtection from './twofactor/SelectProtection.vue';

export default {
  components: {
    SecurePhoneChange,
    SecurityWhitelistedIp,
    SecurityWhitelistedBrowser,
    SecureEmailChange,
    SecureLoginnameChange,
    SecurePasswordChange,
    SelectProtection,
  },

  props: {
    user: {
      type: Object,
    },

    firewallLogs: {
      type: Array,
    },

    security: {
      type: Object,
    },

    xid: {
      type: String,
    },

    authenticatorEnabled: {
      type: Boolean,
      required: false,
    },

    authenticatorDisableUrl: {
      type: String,
    },

    authenticatorEnableUrl: {
      type: String,
    },

    authenticatorQrUrl: {
      type: String,
    },

    whitelistedIps: {
      type: Array,
      required: true,
    },

    whitelistedBrowsers: {
      type: Array,
      required: true,
    },

    loginnameChangeUrl: {
      type: String,
      required: true,
    },

    passwordChangeUrl: {
      type: String,
      required: true,
    },

    phoneChangeUrl: {
      type: String,
      required: true,
    },

    emailChangeUrl: {
      type: String,
      required: true,
    },

    urlWhitelistedIpDelete: {
      type: String,
      required: true,
    },

    urlWhitelistedBrowserDelete: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      tab: null,

      show: {
        emailChange: false,
        passwordChange: false,
        phoneChange: false,
        loginnameChange: false,
      },
    };
  },

  methods: {
    showSection(section) {
      this.show = {
        emailChange: false,
        passwordChange: false,
        phoneChange: false,
        loginnameChange: false,
      };

      this.show[section] = true;
    },
  },
};
</script>
