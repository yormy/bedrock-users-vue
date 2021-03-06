<template>
  <div>
    <div class="card-header">
      {{ $t('bedrock-users.misc.notification.subscription.title') }}
    </div>

    <v-row>
      <v-col> </v-col>
      <v-col> {{ $t('bedrock-users.misc.notification.subscription.email') }} </v-col>
<!--      <v-col> {{ $t('bedrock-users.misc.notification.subscription.slack') }} </v-col>-->
<!--      <v-col> {{ $t('bedrock-users.misc.notification.subscription.sms') }} </v-col>-->
    </v-row>
    <v-row v-for="(template, index) in mailTemplates" :key="index">
      <v-col>
        {{ template.subject }}
      </v-col>
      <v-col>
        <span v-if="template.mailPreventable">
          <v-switch
            v-model="template.mailActive"
            color="success"
            @click="switchNotificationEmail(template)"
            :loading="isLoading === 'MAIL' + template.xid"
            hide-details
            :style="'margin-top: 0px;padding-top: 0px;'"
          ></v-switch>
        </span>
        <span v-else>
          <span class="text-success"><span class="fal fa-check"></span></span>
        </span>
      </v-col>

<!--      <v-col>-->
<!--        <span v-if="template.slackPreventable">-->
<!--          <v-switch-->
<!--            v-model="template.slackActive"-->
<!--            color="success"-->
<!--            hide-details-->
<!--            @click="switchNotificationSlack(template)"-->
<!--            :loading="isLoading === 'SLACK' + template.xid"-->
<!--            :style="'margin-top: 0px;padding-top: 0px;'"-->
<!--          ></v-switch>-->
<!--        </span>-->
<!--        <span v-else>-->
<!--          <span class="text-success"><span class="fal fa-check"></span></span>-->
<!--        </span>-->
<!--      </v-col>-->

<!--      <v-col>-->
<!--        <span v-if="template.smsPreventable">-->
<!--          <v-switch-->
<!--            v-model="template.smsActive"-->
<!--            color="success"-->
<!--            hide-details-->
<!--            @click="switchNotificationSms(template)"-->
<!--            :loading="isLoading === 'SMS' + template.xid"-->
<!--            :style="'margin-top: 0px;padding-top: 0px;'"-->
<!--          ></v-switch>-->
<!--        </span>-->
<!--        <span v-else>-->
<!--          <span class="text-success"><span class="fal fa-check"></span></span>-->
<!--        </span>-->
<!--      </v-col>-->
<!--      -->
    </v-row>
  </div>
</template>

<script>
export default {
  components: {},

  props: {
    title: {
      type: String,
    },

    mailTemplates: {
      type: Array,
      required: true,
    },
  },

  data() {
    return {
      isLoading: null,
    };
  },

  created() {},

  methods: {
    switchNotificationEmail(template) {
      this.switchNotification(template.xid, template.mailActive, 'MAIL');
    },

    switchNotificationSlack(template) {
      this.switchNotification(template.xid, template.slackActive, 'SLACK');
    },

    switchNotificationSms(template) {
      this.switchNotification(template.xid, template.smsActive, 'SMS');
    },

    switchNotification(xid, state, channel) {
      this.isLoading = channel + xid;

      const url = this.route('api.v1.member.account.notifications.settings.subscribe', xid);

      this.$http
        .put(url, { subscribe: state, channel })
        .then(() => {})
        .catch(() => {})
        .finally(() => {
          this.isLoading = null;
        });
    },
  },
};
</script>
