<template>
  <div class="d-flex flex-column">
    <div>
      <h1>Finishing your registration</h1>
      ...nice for promotions
    </div>
    <br />

    <appearing-bullets
      :messages="messages"
      @completed="form.isPreparing = false"
    ></appearing-bullets>

    <div v-if="!form.isPreparing" class="mt-5">
      <button-submit :is-loading="form.isSubmitting" @clicked="goLogin">
        Login
        <template v-slot:loading-title>Logging in... </template>
      </button-submit>
    </div>
  </div>
</template>

<script>
import AppearingBullets from '@components/shared/AppearingBullets.vue';

export default {
  components: {
    AppearingBullets,
  },

  props: {
    registrationXid: String,
    registrationFinishingActionUrl: String,
  },

  data() {
    return {
      form: {
        isPreparing: true,
        isSubmitting: false,
      },

      messages: [
        {
          text: 'Validating your credentials',
          loading: false,
          completed: false,
          processingMs: 3000,
        },
        {
          text: 'Decrypting your data',
          loading: false,
          completed: false,
          processingMs: 2000,
        },
        {
          text: 'Increasing your security level',
          loading: false,
          completed: false,
          processingMs: 1000,
        },
      ],
    };
  },

  mounted() {
    this.finishRegistration();
  },

  methods: {
    goLogin() {
      this.form.isSubmitting = true;
      window.location.replace('/login');
    },

    finishRegistration() {
      this.form.isSubmitting = true;

      const data = {
        xid: this.registrationXid,
      };

      this.$http
        .post(this.registrationFinishingActionUrl, data)
        .then(response => {
          if (response.data.success) {
            this.$emit('finished');
          }
        })
        .finally(() => {
          this.form.isSubmitting = false;
        });
    },
  },
};
</script>
