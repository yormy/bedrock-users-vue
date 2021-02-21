<template>
  <div class="d-flex flex-column">
    <div>
      <h1>{{ $t('bedrock-users.registration.finalizing.title') }}</h1>
      <p>{{ $t('bedrock-users.registration.finalizing.subtitle') }}</p>
    </div>
    <br />

    <appearing-bullets
      :messages="messages"
      @completed="form.isPreparing = false"
    ></appearing-bullets>

    <div v-if="!form.isPreparing" class="mt-5">
      <button-submit :is-loading="form.isSubmitting" @clicked="goLogin">
        {{ $t('bedrock-users.action.login') }}
        <template v-slot:loading-title>{{ $t('bedrock-users.action.logging_in') }} </template>
      </button-submit>
    </div>
  </div>
</template>

<script>
import { AppearingBullets } from 'bedrock-vue-components';

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
          text: this.$t('bedrock-users.registration.finalizing.appearing.message1'),
          loading: false,
          completed: false,
          processingMs: 3000,
        },
        {
          text: this.$t('bedrock-users.registration.finalizing.appearing.message2'),
          loading: false,
          completed: false,
          processingMs: 2000,
        },
        {
          text: this.$t('bedrock-users.registration.finalizing.appearing.message3'),
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
        .then((response) => {
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
