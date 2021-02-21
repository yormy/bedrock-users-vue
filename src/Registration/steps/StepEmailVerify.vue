<template>
  <div class="container">
    <h1>{{ $t('bedrock-users.verify.email.title') }}</h1>

    <div class="card-body">
      <confirm-action
        :confirm-title="confirmableAction.title"
        :confirm-sent-to="sentToEmail"
        :confirm-description="confirmableAction.description"
        :method="confirmableAction.method"
        verify-type="CODE"
        :verify-action-url="route('api.v1.registration.email.verify')"
        :verify-resend-action-url="route('api.v1.registration.email.verify.resend')"
        :xid="confirmableAction.confirmableActionXid"
        @verified="verified"
      >
      </confirm-action>
    </div>
  </div>

</template>

<script>
export default {
  props: {
    confirmableAction: {
      type: Object,
      required: true,
    },

    sentToEmail: {
      type: String,
      required: false,
    },
  },

  data() {
    return {};
  },

  methods: {
    verified() {
      this.$emit('verified');
    },
  },
};
</script>
