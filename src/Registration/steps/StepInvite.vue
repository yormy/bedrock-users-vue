<template>
  <div class="container">
    <h1 v-if="'OPTIONAL' === inviteCodeSetting ">
      {{$t('bedrock-users.registration.input.invite_code.title_optional')}}
    </h1>
    <h1 v-else>
      {{$t('bedrock-users.registration.input.invite_code.title_required')}}
    </h1>

    <CodeInput
      :loading="form.isSubmitting"
      ref="codeInputComponent"
      class="input"
      type="string"
      :fields="5"
      @change="onCodeChange"
      @complete="onCodeComplete"
      @enter="onCodeEnter"
    />
    <div v-if="apiErrors.inviteCode" class="alert alert-danger">
      {{ apiErrors.inviteCode }}
    </div>

    <div class="mt-3">

      <button
        type="submit"
        v-if="skippableInviteCode"
        class="btn btn-secondary"
        :disabled="form.isSubmitting"
        @click.prevent="inviteSkipAction"
      >
        {{ $t('bedrock-users.action.skip') }}
      </button>

      <button-submit :is-loading="form.isSubmitting" @clicked="inviteAction">
        {{ $t('bedrock-users.action.next') }}
      </button-submit>

    </div>
  </div>
</template>

<script>
import { getFirstValidationError } from 'bedrock-vue-components';
import { CodeInput } from 'bedrock-vue-components';

export default {
  components: {
    CodeInput,
  },

  props: {
    registrationInviteActionUrl: String,
    inviteCodeSetting: String,
    registrationXid: String,
  },

  data() {
    return {
      form: {
        isSubmitting: false,
      },

      invitecode: {
        value: null,
        error: '',
      },

      inviteCode: null,
      inviteCodeError: null,

      apiErrors: {},
    };
  },

  computed: {
    skippableInviteCode() {
      return this.inviteCodeSetting === 'OPTIONAL';
    },
  },

  methods: {
    onCodeChange() {
      this.inviteCode = null;
    },

    onCodeComplete(value) {
      this.inviteCode = value;
    },

    onCodeEnter() {
      this.inviteAction();
    },

    inviteAction() {
      this.form.isSubmitting = true;

      const data = {
        xid: this.registrationXid,
        inviteCode: this.inviteCode,
      };

      this.$http
        .post(this.registrationInviteActionUrl, data)
        .then((response) => {
          if (response.data.success) {
            this.inviteCodeError = '';
            this.$emit('confirmed');
          }

          this.form.isSubmitting = false;
        })
        .catch((error) => {
          this.apiErrors.inviteCode = error.response.data.message;

          const inviteCodeError = getFirstValidationError(error, 'inviteCode');
          if (inviteCodeError !== '') {
            this.apiErrors.inviteCode = inviteCodeError;
          }
          this.$refs.codeInputComponent.clear();
          this.form.isSubmitting = false;
        });
    },

    inviteSkipAction() {
      this.$emit('skipped');
    },
  },
};
</script>
