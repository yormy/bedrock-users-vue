<template>
  <div>
    <div v-if="!forgot">
      <login
        :login-action-url="loginActionUrl"
        :token-ip-whitelisting="tokenIpWhitelisting"
        :token-error-message="tokenErrorMessage"
        :loginas="loginas"
        @submitting="submitting"
      >
      </login>

      <div v-if="!isSubmitting">
        <div v-if="registrationActionUrl">
          <div class="d-flex justify-content-center">
            <div class="d-flex justify-content-end">
              <button id="btnRegistration">
                {{ $t('misc.login.no_account_signup.title') }}
                <a :href="registrationActionUrl">
                  {{ $t('misc.login.no_account_signup.buttontext') }}
                </a>
              </button>
            </div>
          </div>
        </div>

        <div class="d-flex justify-content-end">
          <button id="btnShowForgot" @click="forgot = true">
            <span class="fal fa-unlock mr-1"></span>
            {{ $t('misc.login.forgot.title') }}
          </button>
        </div>
      </div>
    </div>

    <div v-else>
      <div class="btn btn-primary" @click="showPasswordForgot = true">
        {{ $t('misc.password') | capitalizeFirst }}
      </div>
      <div class="btn btn-primary" @click="showPasswordForgot = false">
        {{ $t('misc.loginname') | capitalizeFirst }}
      </div>

      <div v-if="showPasswordForgot">
        <h1>{{ $t('profile.reset.password.title') | capitalizeFirst }}</h1>
        <reset-password :reset-password-request-url="resetPasswordRequestUrl"> </reset-password>
      </div>
      <div v-else>
        <h1>{{ $t('profile.reset.loginname.title') | capitalizeFirst }}</h1>
        <reset-loginname :reset-loginname-request-url="resetLoginnameRequestUrl"> </reset-loginname>
      </div>

      <div class="d-flex justify-content-end">
        <button id="btnShowLogin" @click="forgot = false">
          <span class="fal fa-unlock mr-1"></span>Login
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import Login from './Login.vue';
import ResetLoginname from './ResetLoginname.vue';
import ResetPassword from './ResetPassword.vue';

export default {
  components: {
    ResetLoginname,
    ResetPassword,
    Login,
  },

  props: {
    loginActionUrl: {
      type: String,
      required: true,
    },

    registrationActionUrl: {
      type: String,
      required: false,
    },

    resetLoginnameRequestUrl: {
      type: String,
      required: true,
    },

    resetPasswordRequestUrl: {
      type: String,
      required: true,
    },

    tokenIpWhitelisting: {
      type: String,
      required: false,
    },

    tokenErrorMessage: {
      type: String,
      required: false,
    },

    loginas: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      forgot: false,
      isSubmitting: false,

      showPasswordForgot: true,
    };
  },

  methods: {
    submitting(isSubmitting) {
      this.isSubmitting = isSubmitting;
    },
  },
};
</script>
