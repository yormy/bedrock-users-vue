<template>
  <div class="container">
    <confirm-action
      :confirm-title="confirmableAction.title"
      :confirm-sent-to="`Sent to ${sentToPhone}`"
      :confirm-description="confirmableAction.description"
      :method="confirmableAction.method"
      verify-type="CODE"
      :verify-action-url="route('api.v1.user.registration.verify.verify')"
      :verify-resend-action-url="route('api.v1.user.registration.resend')"
      :xid="confirmableAction.confirmableActionXid"
      @verified="verified"
    >
    </confirm-action>

    <button @click="changePhone">Click to Change phone</button>
  </div>
</template>

<script>
export default {
  props: {
    confirmableAction: {
      type: Object,
      required: true,
    },

    sentToPhone: {
      type: String,
      required: false,
    },
  },

  data() {
    return {};
  },

  methods: {
    changePhone() {
      this.$emit('changePhone');
    },

    verified() {
      this.$emit('verified');
    },
  },
};
</script>
