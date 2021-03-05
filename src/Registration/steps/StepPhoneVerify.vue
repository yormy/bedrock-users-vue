<template>
  <div class="card-body">

    <confirm-action
      :confirm-title="confirmableAction.title"
      :confirm-sent-to="sentToPhone"
      :confirm-description="confirmableAction.description"
      :method="confirmableAction.method"
      verify-type="CODE"
      :verify-action-url="route('api.v1.registration.phone.verify')"
      :verify-resend-action-url="route('api.v1.registration.phone.verify.resend')"
      :xid="confirmableAction.xid"
      @verified="verified"
    >
    </confirm-action>

    <div class="mt-3">
      <button @click="changePhone">{{ $t('bedrock-users.registration.click_to_change_phone') }}</button>
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
