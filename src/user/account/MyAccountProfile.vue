<template>
  <div>
    <v-tabs
      v-model="tab"
      background-color="bg-info"
      class="elevation-2"
      dark
      :grow="true"
      :icons-and-text="true"
    >
      <v-tabs-slider></v-tabs-slider>

      <v-tab :href="`#tab-account`">
        {{ $t('bedrock-users.profile.account') }}
        <i class="fal fa-user-circle fa-2x"></i>
      </v-tab>

      <v-tab :href="`#tab-statistics`">
        {{ $t('bedrock-users.profile.statistics') }}
        <i class="fal fa-chart-scatter fa-2x"></i>
      </v-tab>

      <v-tab-item :value="'tab-account'">
        <my-account-profile-details
          :user="user"
          :timezones="timezones"
          :update-account-url="updateAccountUrl"
        ></my-account-profile-details>
      </v-tab-item>

      <v-tab-item :value="'tab-statistics'">
        <my-account-profile-statistics
          :user="user"
        >

        </my-account-profile-statistics>
      </v-tab-item>
    </v-tabs>
  </div>
</template>

<script>
import MyAccountProfileDetails from './MyAccountProfileDetails.vue';
import MyAccountProfileStatistics from "./MyAccountProfileStatistics.vue";

export default {
  components: {
    MyAccountProfileStatistics,
    MyAccountProfileDetails,
  },

  props: {
    // permissions: {
    //   type: Array,
    // },

    user: {
      type: Object,
    },

    timezones: {
      type: Array,
      required: true,
    },

    statistics: {
      type: Object,
    },

    updateAccountUrl: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      tab: null,

      form: {
        timezone: this.user.timezone,
      },
    };
  },
};
</script>
