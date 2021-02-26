<template>
  <div class="container">
    <h1>enter your phone</h1>
    Why we need your phone... We need your phone number to be able to verify your identity, after
    which you will be granted access to our portal
    <maz-phone-number-input
      v-model="phoneNumerUnformatted"
      size="lg"
      valid-color="#459B19"
      error-color="#FF0000"
      :required="true"
      @update="updated"
      :default-country-code="'NL'"
      @keyup.native.enter="registerAction"
    />

    <div v-if="apiErrors.phone">
      <div class="alert alert-danger" style="overflow-wrap: break-word">
        <strong>{{ apiErrors.phone }}</strong>
      </div>
    </div>

    <div v-if="apiErrors.xid">
      <div class="alert alert-danger" style="overflow-wrap: break-word">
        <strong>{{ apiErrors.xid }}</strong>
      </div>
    </div>

    <div v-if="apiErrors.form">
      <div class="alert alert-danger" style="overflow-wrap: break-word">
        <strong>{{ apiErrors.form }}</strong>
      </div>
    </div>

    <button-submit :is-loading="form.isSubmitting" @clicked="registerAction"> Next </button-submit>
  </div>
</template>

<script>
import { mergeErrors } from 'bedrock-vue-components';

export default {
  props: {
    registrationXid: String,
    registrationPhoneActionUrl: String,
  },

  data() {
    return {
      form: {
        isSubmitting: false,
      },

      phoneNumerUnformatted: '',

      registration: {
        phone: '',
        xid: '',
      },

      apiErrors: {
        form: '',
      },

      phoneValid: false,
    };
  },
  mounted() {
    this.registration.xid = this.registrationXid;
  },
  methods: {
    updated(data) {
      this.phoneValid = data.isValid;

      if (data.isValid) {
        this.registration.phone = data.e164;
      }
      this.apiErrors.form = '';
    },

    registerAction() {
      if (!this.phoneValid) {
        this.apiErrors.form = this.$t('misc.validations.phone.invalid');
        return;
      }

      this.form.isSubmitting = true;

      const data = {
        phone: this.registration.phone,
        xid: this.registration.xid,
      };

      this.$http
        .post(this.registrationPhoneActionUrl, data)
        .then((response) => {
          if (response.data.success) {
            this.isSubmitting = false;
            this.$emit('finished', response.data.data);
          }
        })
        .catch((error) => {
          this.apiErrors = mergeErrors(error);
        })
        .finally(() => {
          this.form.isSubmitting = false;
        });
    },
  },
};
</script>
