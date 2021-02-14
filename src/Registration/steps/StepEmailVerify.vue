<template>
  <div class="container">
    <h1>Verify your email addresss</h1>
    <div class="card-body">
      <confirm-action
        :confirm-title="confirmableAction.title"
        :confirm-sent-to="`Sent to ${sentToEmail}`"
        :confirm-description="confirmableAction.description"
        :method="confirmableAction.method"
        :resend-possible="false"
        verify-type="CODE"
        :verify-action-url="route('api.v1.user.registration.verify.verify')"
        :verify-resend-action-url="route('api.v1.user.registration.resend')"
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
