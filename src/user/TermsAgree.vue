<template>
  <div class="container">
    <h1>{{ $t('bedrock-users.profile.terms.agree_to_terms') }}</h1>

    <ValidationProvider
      v-show="!generalAgreed"
      v-slot="{ errors }"
      rules="agreed"
      name="termsGeneral"
      ref="termsGeneral"
    >
      <div>
        {{ $t('bedrock-users.profile.terms.statement.general') }}
        <a :href="route('guest.terms.general')" target="_blank"><span class="fal fa-external-link">&nbsp;</span></a>
        <v-checkbox
          v-model="form.termsGeneral"
          class="mt-0"
          :label="$t('bedrock-users.profile.terms.i_agree_to_above')"
          :error-messages="errors"
        >
        </v-checkbox>
      </div>
    </ValidationProvider>

    <ValidationProvider
      v-show="!marketingAgreed"
      name="termsMarketing"
      rules="agreed"
      ref="termsMarketing"
    >
      {{ $t('bedrock-users.profile.terms.statement.marketing') }}
      <a :href="route('guest.terms.marketing')"  target="_blank"><span class="fal fa-external-link">&nbsp;</span></a>
      <div>
        <v-checkbox
          v-model="form.termsMarketing"
          class="mt-0"
          :label="$t('bedrock-users.misc.terms.i_agree_to_above')"
          :error-messages="form.errorMessage"
        >
          <template v-slot:label>
            <div :class="marketing.errorMessage !== '' ? 'red--text' : ''">
              {{ $t('bedrock-users.profile.terms.i_agree_to_above') }}
            </div>
            <div v-if="marketing.errorMessage" class="red--text">
              &nbsp; == {{ marketing.errorMessage }}
            </div>
          </template>
        </v-checkbox>
      </div>
    </ValidationProvider>

    <div v-if="state.general.errorMessage" class="alert alert-danger">
      {{ state.general.errorMessage }}
    </div>

    <button-submit :is-loading="state.isSubmitting" @clicked="termsAgreeAction">
      {{ $t('bedrock-users.profile.terms.agreed') }}
    </button-submit>
  </div>
</template>

<script>

export default {
  props: {
    termsAgreeActionUrl: String,
    generalAgreed: Number,
    marketingAgreed: Number,
  },

  data() {
    return {
      state: {
        isSubmitting: false,
        general: {
          successMessage: '',
          errorMessage: '',
        },

        marketing: {
          successMessage: '',
        },

        successMessageMarketing: '',
        errorMessage: '',
      },

      form: {
        termsGeneral: true,
        termsMarketing: true,
      },

      marketing: {
        errorMessage: '',
        isRetry: false,
      },

      lang: {
        error: {
          marketing: this.$t('bedrock-users.profile.terms.marketing_push'),
        },
      },
    };
  },

  created() {
    this.form.termsGeneral = this.generalAgreed;
    this.form.termsMarketing = this.marketingAgreed;
  },

  methods: {
    isFormValid() {
      let formValid = true;
      this.$refs.termsGeneral.validate();
      if (!this.marketing.isRetry) {
        this.$refs.termsMarketing.validate();
      }

      // prevent because terms are not agreed
      if (!this.form.termsGeneral) {
        formValid = false;
      }

      // user can skip this, but not on the first click
      if (!this.form.termsMarketing && !this.marketing.isRetry) {
        this.marketing.errorMessage = this.lang.error.marketing;
        formValid = false;
      }

      this.marketing.isRetry = true;
      return formValid;
    },

    termsAgreeAction() {
      if (!this.isFormValid()) {
        return;
      }

      this.state.isSubmitting = true;
      this.state.general.errorMessage = '';

      const data = {
        general: this.form.termsGeneral,
        marketing: this.form.termsMarketing,
      };

      this.$http
        .post(this.termsAgreeActionUrl, data)
        .then(response => {
          if (response.data.success) {
            // this.state.general.successMessage = response.data.data.general;
            // this.state.marketing.successMessage = response.data.data.marketing;
            const urlParams = new URLSearchParams(window.location.search);
            const intendedUrl = atob(urlParams.get('intendedUrl'));
            window.location.replace(intendedUrl);
          }
        })
        .catch(error => {
          this.state.general.errorMessage = error.response.data.message;
        })
        .finally(() => {
          this.state.isSubmitting = false;
        });
    },
  },
};
</script>

<style scoped></style>
