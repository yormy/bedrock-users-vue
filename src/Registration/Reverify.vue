<template>
  <step-email-verify
    :confirmable-action="confirmableAction"
    :sent-to-email="sentToEmail"
  ></step-email-verify>
<!--  @verified="verifiedEmail"-->

</template>

<script>
import StepEmailVerify from './steps/StepEmailVerify.vue';

export default {
  components: {
    StepEmailVerify,
  },

  props: {
    confirmableAction: {
      type: Object,
      required: true
    },

    sentToEmail: {
      type: String,
      required: true
    },
  }
}
</script>
