<template>
  <div>
    <v-tabs
      v-model="tab"
      background-color="bg-success"
      class="elevation-2"
      dark
      :grow="true"
      :icons-and-text="true"
    >
      <v-tabs-slider></v-tabs-slider>

      <v-tab :href="`#tab-emails`">
        {{ $t('bedrock-users.profile.emails')}}
        <i class="fal fa-envelope fa-2x"></i>
      </v-tab>

      <v-tab :href="`#tab-sessions`">
        {{ $t('bedrock-users.profile.sessions') }}

        <i class="fal fa-user-friends fa-2x"></i>
      </v-tab>

      <v-tab-item :value="'tab-emails'">
        <activity-emails :datatableValues="sentEmails" :open-email-url="openEmailUrl">
        </activity-emails>
      </v-tab-item>

      <v-tab-item :value="'tab-sessions'">
        <activity-sessions
          :datatableValues="activeSessions"
          :current-session="currentSession"
          :delete-session-url="deleteSessionUrl"
          :delete-all-other-sessions-url="deleteAllOtherSessionsUrl"
        >
        </activity-sessions>
      </v-tab-item>
    </v-tabs>
  </div>
</template>

<script>
import ActivityEmails from './profile/ActivityEmails.vue';
import ActivitySessions from './profile/ActivitySessions.vue';

export default {
  components: {
    ActivitySessions,
    ActivityEmails,
  },

  props: {
    sentEmails: {
      type: Array,
      required: true,
    },

    activities: {
      type: Array,
    },

    agreements: {
      type: Array,
    },

    audits: {
      type: Array,
    },

    openEmailUrl: {
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
  },

  data() {
    return {
      tab: null,
    };
  },
};
</script>
