<template>
  <div>
    <loading-overlay :show="state.isSubmitting"></loading-overlay>
    <div class="col-12 text-center">
      {{ $t('bedrock-users.profile.no-available-statistics') }}
    </div>


  </div>
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      required: true,
    },

  },

  data() {
    return {
      state: {
        isSubmitting: false,
      },
    };
  },

};
</script>
