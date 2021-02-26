<template>
  <div>
    <loading-overlay :show="state.isSubmitting"></loading-overlay>
    <div class="col-4">

      <v-text-field
        :label="$t('bedrock-users.user.email')"
        v-model="user.email"
        disabled
      ></v-text-field>

      <v-text-field
        :label="$t('bedrock-users.user.name')"
        v-model="user.name"
        disabled
      ></v-text-field>

      <v-text-field
        :label="$t('bedrock-users.user.phone')"
        v-model="user.phone"
        disabled
      ></v-text-field>

      <v-autocomplete
        :label="$t('bedrock-users.user.timezone.title')"
        v-model="form.timezone"
        :hint="$t('bedrock-users.user.timezone.hint')"
        :items="timezones"
        persistent-hint
      >
      </v-autocomplete>

      <div class="my-3 float-right">
        <button-submit :is-loading="state.isSubmitting" @clicked="saveAction">
          {{ $t('bedrock-users.action.save') }}
        </button-submit>
      </div>
    </div>


  </div>
</template>

<script>
import { mergeErrors } from 'bedrock-vue-components';

export default {
  props: {
    user: {
      type: Object,
      required: true,
    },

    timezones: {
      type: Array,
      required: true,
    },

    updateAccountUrl: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      tab: null,

      state: {
        isSubmitting: false,
      },

      form: {
        timezone: this.user.timezone,
      },
    };
  },

  methods: {
    saveAction() {
      const data = {
        timezone: this.form.timezone,
      };

      this.startLoadingState();

      this.$http
        .post(this.updateAccountUrl, data)
        .then(response => {
          if (response.data.success) {
            this.state.successMessage = response.data.data;
          }
        })
        .catch(error => {
          this.apiErrors = mergeErrors(error);
        })
        .finally(() => {
          this.stopLoadingState();
        });
    },

    startLoadingState() {
      this.state.isSubmitting = true;
    },

    stopLoadingState() {
      this.state.isSubmitting = false;
    },
  },
};
</script>
