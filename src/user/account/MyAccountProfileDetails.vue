<template>
  <div>
    <loading-overlay :show="state.isSubmitting"></loading-overlay>
    <div class="row">
      <div class="col-4">

<!--        <v-text-field-->
<!--          :label="$t('bedrock-users.user.email')"-->
<!--          v-model="user.email"-->
<!--          disabled-->
<!--        ></v-text-field>-->

        <div class="my-3">
          <div>
            {{$t('bedrock-users.user.email')}}:
            <span @click="showSection('emailChange')" class="fal fa-pencil float-right"></span>
          </div>
          <div>
            {{user.email}}

          </div>
        </div>

        <div class="my-3">
          <div>
            {{$t('bedrock-users.user.loginname.label')}}:
            <span @click="showSection('loginnameChange')" class="fal fa-pencil float-right"></span></div>
          <div>
            ********
          </div>
        </div>

        <div class="my-3">
          <div>
            {{$t('bedrock-users.user.phone')}}:
            <span @click="showSection('phoneChange')" class="fal fa-pencil float-right"></span>
          </div>
          <div>
            <span v-if="user.phone">{{user.phone}}</span>
            <span v-else>-</span>
          </div>
        </div>

        <div>
          {{$t('bedrock-users.user.password')}}:
          <span @click="showSection('passwordChange')" class="fal fa-pencil float-right"></span>
        </div>
        <div>
          ********
        </div>

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

      <div class="col-8">
          <div v-if="show.phoneChange">
            <secure-phone-change
              :change-action-url="phoneChangeUrl"
              :authenticator-enabled="authenticatorEnabled"
            >
            </secure-phone-change>
          </div>

          <div v-if="show.emailChange">
            <secure-email-change
              :user="user"
              :change-action-url="emailChangeUrl"
              :authenticator-enabled="authenticatorEnabled"
            >
            </secure-email-change>
          </div>

          <div v-if="loginnameChangeUrl">
            <div v-if="show.loginnameChange">
              <secure-loginname-change
                :user="user"
                :change-action-url="loginnameChangeUrl"
                :authenticator-enabled="authenticatorEnabled"
              >
              </secure-loginname-change>
            </div>
          </div>

          <div v-if="loginnameChangeUrl">
            <div v-if="show.passwordChange">
              <secure-password-change
                :user="user"
                :change-action-url="passwordChangeUrl"
                :authenticator-enabled="authenticatorEnabled"
              >
              </secure-password-change>
            </div>
          </div>
        </div>
      </div>
  </div>


  </div>
</template>

<script>
import { mergeErrors } from 'bedrock-vue-components';

import SecureEmailChange from './profile/SecureEmailChange.vue';
import SecurePhoneChange from './profile/SecurePhoneChange.vue';
import SecureLoginnameChange from './profile/SecureLoginnameChange.vue';
import SecurePasswordChange from './profile/SecurePasswordChange.vue';

export default {
  components: {
    SecurePhoneChange,
    SecureEmailChange,
    SecureLoginnameChange,
    SecurePasswordChange,
  },

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

    loginnameChangeUrl: {
      type: String,
      required: true,
    },

    passwordChangeUrl: {
      type: String,
      required: true,
    },

    phoneChangeUrl: {
      type: String,
      required: true,
    },

    emailChangeUrl: {
      type: String,
      required: true,
    },

    authenticatorEnabled: {
      type: Boolean,
      required: false,
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

      show: {
        emailChange: false,
        passwordChange: false,
        phoneChange: false,
        loginnameChange: false,
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

    showSection(section) {
      this.show = {
        emailChange: false,
        passwordChange: false,
        phoneChange: false,
        loginnameChange: false,
      };

      this.show[section] = true;
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
