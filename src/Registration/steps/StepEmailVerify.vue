<template>
  <div class="card-body">
    {{confirmableAction}}
    <confirm-action
      :confirm-title="confirmableAction.title"
      :confirm-sent-to="sentToEmail"
      :confirm-description="confirmableAction.description"
      :method="confirmableAction.method"
      verify-type="CODE"
      :verify-action-url="route('api.v1.registration.email.verify')"
      :verify-resend-action-url="route('api.v1.registration.email.verify.resend')"
      :xid="confirmableAction.xid"
      @verified="verified"
    >
    </confirm-action>
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
