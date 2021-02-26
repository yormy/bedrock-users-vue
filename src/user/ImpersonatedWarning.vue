<template>
  <div v-if="impersonated" class="alert alert-danger">
    <div class="text-center">
      <strong>{{ $t('profile.impersonated') | capatalize }}</strong>
      <button-submit
        :is-loading="state.isSubmitting"
        btn-class="btn btn-success"
        @clicked="leaveImpersonation"
      >
        {{ $t('profile.impersonated_leave') | capatalize }}
      </button-submit>
    </div>
  </div>
</template>

<script>
import { removeTokensUser } from 'bedrock-vue-components';

export default {
  computed: {
    impersonated() {
      return localStorage.getItem('impersonated');
    },
  },

  data() {
    return {
      state: {
        isSubmitting: false,
      },
    };
  },

  methods: {
    leaveImpersonation() {
      const url = this.route('admin.users.impersonate.leave');

      this.state.isSubmitting = true;
      this.$http
        .get(url)
        .then(response => {
          if (response.data.success) {
            removeTokensUser();
            localStorage.removeItem('impersonated');
            window.location.reload();
          }
        })
        .catch(() => {})
        .finally(() => {});
    },
  },
};
</script>

<style scoped>
.alert {
  z-index: 99 !important;
}
</style>
