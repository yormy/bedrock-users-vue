<template>
  <div>
    <v-row>
      <v-col>
        {{ user.name }}

        <a :href="route('user.account.billing.index')">
          <v-chip v-if="user.current_plan" :color="user.current_plan.color" x-small outlined
            >{{ user.current_plan.name }}
            {{ user.current_plan.billing_period_shorthand }}
          </v-chip>
        </a>
      </v-col>
      <v-col>
        {{ user.email }}
      </v-col>
      <v-col>
        <div v-if="authenticatorEnabled"> {{ $t('bedrock-users.profile.security.authenticator_enabled') }} </div>
        <div v-else> {{ $t('bedrock-users.profile.security.whitelisting_enabled') }}</div>
      </v-col>
    </v-row>
  </div>
</template>

<script>
export default {
  props: {
    title: {
      type: String,
    },

    user: {
      type: Object,
      required: true,
    },

    authenticatorEnabled: {
      type: Boolean,
      required: true,
    },
  },
};
</script>
