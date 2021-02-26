<template>
  <div class="container">
    <h1>Terms reset</h1>
    <!--    <textarea-->
    <!--      id="summary_english"-->
    <!--      class="form-control"-->
    <!--      rows="5"-->
    <!--      v-model="form.general_terms_summary"-->
    <!--    ></textarea>-->

    <div v-if="state.successMessage.GENERAL" class="alert alert-success">
      <li v-for="(item, key, index) in state.successMessage.GENERAL" :key="index">
        {{ key }} : {{ item }}
      </li>
    </div>

    <div v-if="state.successMessage.MARKETING" class="alert alert-success">
      <li v-for="(item, key, index) in state.successMessage.MARKETING" :key="index">
        {{ key }} : {{ item }}
      </li>
    </div>

    <button-submit :is-loading="state.isSubmitting" @clicked="resetTermsAction">
      Reset
    </button-submit>
  </div>
</template>

<script>
import { mergeErrors } from 'bedrock-vue-components';

export default {
  props: {
    resetActionUrl: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      state: {
        isSubmitting: false,

        successMessage: {},

        successMessageMarketing: '',
        errorMessage: '',
      },

      form: {
        general_terms_summary: '',
      },
    };
  },

  methods: {
    resetTermsAction() {
      this.state.isSubmitting = true;

      const data = {
        general_terms_summary: this.general_terms_summary,
      };

      this.$http
        .post(this.resetActionUrl, data)
        .then(response => {
          if (response.data.success) {
            this.state.successMessage = response.data.data;
          }
        })
        .catch(error => {
          this.apiErrors = mergeErrors(error);
        })
        .finally(() => {
          this.state.isSubmitting = false;
        });
    },
  },
};
</script>

<style scoped></style>
