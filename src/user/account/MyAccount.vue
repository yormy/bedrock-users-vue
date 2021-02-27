<template>
  <div>
    <div class="card-header">
      {{ $t('bedrock-users.profile.user_details') }}
    </div>
    <v-row>
      <v-col>
        <my-account-overview :user="user" :authenticator-enabled="authenticatorEnabled"></my-account-overview>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <performance-card
          :title="$t('bedrock-users.profile.tabs.details.title')"
          :subtitle="$t('bedrock-users.profile.tabs.details.description')"
          :color="'bg-info'"
          :clickable="true"
          @clicked="section = 'PROFILE'"
        >
        </performance-card>
      </v-col>
      <v-col>
        <performance-card
          :title="$t('bedrock-users.profile.tabs.activity.title')"
          :subtitle="$t('bedrock-users.profile.tabs.activity.description')"
          :color="'bg-success'"
          :clickable="true"
          @clicked="section = 'ACTIVITY'"
        >
        </performance-card>
      </v-col>
      <v-col>
        <performance-card
          :title="$t('bedrock-users.profile.tabs.security.title')"
          :subtitle="$t('bedrock-users.profile.tabs.security.description')"
          :color="'bg-warning'"
          :clickable="true"
          @clicked="section = 'SECURITY'"
        >
        </performance-card>
      </v-col>
    </v-row>

    <div v-if="section == 'PROFILE'">
      <my-account-profile
        :user="user"
        :timezones="timezones"
        :update-account-url="updateAccountUrl"
        :loginname-change-url="loginnameChangeUrl"
        :password-change-url="passwordChangeUrl"
        :phone-change-url="phoneChangeUrl"
        :email-change-url="emailChangeUrl"
        :authenticator-enabled="authenticatorEnabled"
      ></my-account-profile>

      <!--        :user="user" :permissions="permissions" :statistics="statistics"-->
    </div>

    <div v-if="section == 'ACTIVITY'">
      <my-account-activity
        :sent-emails="sentEmails"
        :open-email-url="openEmailUrl"
        :active-sessions="activeSessions"
        :current-session="currentSession"
        :delete-session-url="deleteSessionUrl"
        :delete-all-other-sessions-url="deleteAllOtherSessionsUrl"
      >
      </my-account-activity>
    </div>

    <div v-if="section == 'SECURITY'">
      <my-account-security
        :user="user"
        :whitelisted-ips="whitelistedIps"
        :whitelisted-browsers="whitelistedBrowsers"
        :authenticator-enabled="authenticatorEnabled"
        :authenticator-disable-url="authenticatorDisableUrl"
        :authenticator-enable-url="authenticatorEnableUrl"
        :authenticator-qr-url="authenticatorQrUrl"
        :loginname-change-url="loginnameChangeUrl"
        :password-change-url="passwordChangeUrl"
        :phone-change-url="phoneChangeUrl"
        :email-change-url="emailChangeUrl"
        :url-whitelisted-ip-delete="urlWhitelistedIpDelete"
        :url-whitelisted-browser-delete="urlWhitelistedBrowserDelete"
      ></my-account-security>
    </div>
  </div>
</template>

<script>
import MyAccountOverview from './MyAccountOverview.vue';
import MyAccountProfile from './MyAccountProfile.vue';
import MyAccountActivity from './MyAccountActivity.vue';
import MyAccountSecurity from './MyAccountSecurity.vue';

export default {
  components: {
    MyAccountOverview,
    MyAccountProfile,
    MyAccountActivity,
    MyAccountSecurity,
  },

  props: {
    title: {
      type: String,
    },

    user: {
      type: Object,
      required: true,
    },

    timezones: {
      type: Array,
      required: true,
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

    sentEmails: {
      type: Array,
      required: true,
    },

    openEmailUrl: {
      type: String,
      required: true,
    },

    updateAccountUrl: {
      type: String,
      required: true,
    },

    activeSessions: {
      type: Array,
      required: true,
    },

    currentSession: {
      type: String,
      required: true,
    },

    deleteSessionUrl: {
      type: String,
      required: true,
    },

    deleteAllOtherSessionsUrl: {
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
      section: 'PROFILE',
    };
  },

  created() {},

  methods: {
    openUser(item) {
      const url = this.urlOpenUser.replace('XID', item.xid);
      window.location.href = url;
    },

    hasFilter() {
      if (this.roleSelected && this.roleSelected.key) {
        return true;
      }
      return false;
    },

    applyFilters(item) {
      if (this.roleSelected.key === 'no-roles') {
        return !item.roles.length;
      }

      const found = item.roles.find(function fn(element) {
        return element.name === this.roleSelected.key;
      }, this);

      return found;
    },
  },
};
</script>
