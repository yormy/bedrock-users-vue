<template>
  <div class="container">
    <div v-if="successMessage" class="alert alert-success">
      {{ successMessage }}
    </div>

    <div v-if="error" class="alert alert-danger">
      {{ error }}
    </div>

    <div v-if="registerStep === REGISTER_STEP_EMAIL">
      <step-email
        :registration-action-url="registrationActionUrl"
        @finished="registrationStepEmailEntered"
      ></step-email>
    </div>

    <div v-if="registerStep === REGISTER_STEP_INVITE">
      <step-invite
        :registration-xid="registrationXid"
        :registration-invite-action-url="registrationInviteActionUrl"
        :invite-code-setting="inviteCodeSetting"
        @confirmed="registrationStepInviteConfirmed"
        @skipped="registrationStepInviteSkipped"
      ></step-invite>
    </div>

    <div v-if="registerStep === REGISTER_STEP_PHONE">
      <step-phone
        :registration-phone-action-url="registrationPhoneActionUrl"
        :registration-xid="registrationXid"
        @finished="registrationStepPhoneEntered"
      >
      </step-phone>
    </div>

    <div v-if="registerStep === REGISTER_STEP_EMAIL_VERIFY">
      <step-email-verify
        :confirmable-action="confirmableAction"
        :sent-to-email="sentToEmail"
        @verified="verifiedEmail"
      ></step-email-verify>
    </div>

    <div v-if="registerStep === REGISTER_STEP_PHONE_VERIFY">
      <step-phone-verify
        :confirmable-action="confirmableAction"
        :sent-to-phone="sentToPhone"
        @verified="verifiedPhone"
        @changePhone="changePhone"
      >
      </step-phone-verify>
    </div>

    <div v-if="registerStep === REGISTER_STEP_FINISHED">
      <step-finished
        :registration-xid="registrationXid"
        :registration-finishing-action-url="registrationFinishingActionUrl"
      >
      </step-finished>
    </div>
  </div>
</template>

<script>
import StepEmail from './steps/StepEmail.vue';
import StepInvite from './steps/StepInvite.vue';
import StepEmailVerify from './steps/StepEmailVerify.vue';
import StepPhone from './steps/StepPhone.vue';
import StepPhoneVerify from './steps/StepPhoneVerify.vue';
import StepFinished from './steps/StepFinished.vue';

export default {
  components: {
    StepEmail,
    StepInvite,
    StepPhone,
    StepPhoneVerify,
    StepEmailVerify,
    StepFinished,
  },

  props: {
    error: String,
    continueRegistrationXid: String,
    registrationActionUrl: String,
    registrationPhoneActionUrl: String,
    registrationInviteActionUrl: String,
    registrationFinishingActionUrl: String,
    inviteCodeSetting: String,
    phoneMandatory: null,

    currentStep : {
      type: String,
      required : true
    },

    action : {
      type: Object,
      required : false
    },

    sentToEmailProp : {
      type: String,
      required : false
    },

    sentToPhoneProp : {
      type: String,
      required : false
    }
  },

  data() {
    return {
      registerStep: 1,
      steps: [],
      registrationXid: '',
      successMessage: '',

      sentToEmail: this.sentToEmailProp,
      sentToPhone: this.sentToPhoneProp,

      confirmableAction: {
        xid: '',
        method: '',
        title: '',
        description: '',
      },

    };
  },

  computed: {
    REGISTER_STEP_EMAIL: () => 1,
    REGISTER_STEP_INVITE: () => 2,
    REGISTER_STEP_EMAIL_VERIFY: () => 3,
    REGISTER_STEP_PHONE: () => 4,
    REGISTER_STEP_PHONE_VERIFY: () => 5,
    REGISTER_STEP_FINISHED: () => 99,
  },

  mounted() {
    this.steps = [this.REGISTER_STEP_EMAIL];

    if (this.inviteCodeSetting !== 'NONE') {
      this.steps.push(this.REGISTER_STEP_INVITE);
    }

    this.steps.push(this.REGISTER_STEP_EMAIL_VERIFY);

    if (this.phoneMandatory) {
      this.steps.push(this.REGISTER_STEP_PHONE);
    }

    if (this.phoneMandatory) {
      this.steps.push(this.REGISTER_STEP_PHONE_VERIFY);
    }

    this.steps.push(this.REGISTER_STEP_FINISHED);

    if (this.continueRegistrationXid) {

      if (this.currentStep ==='EMAIL_VERIFY') {
        this.registrationXid = this.continueRegistrationXid;
        this.registerStep = this.REGISTER_STEP_EMAIL_VERIFY;

        this.confirmableAction.xid = this.action.xid;
        this.confirmableAction = this.action;
        this.confirmableAction.method = 'EMAIL';
      }
      //
      //
      // this.successMessage = 'Email verified successfully';
      // this.gotoNext();
    }
  },

  methods: {
    gotoNext() {
      const currentStepIndex = this.steps.findIndex((element) => element === this.registerStep);
      if (this.steps.length > currentStepIndex + 1) {
        this.registerStep = this.steps[currentStepIndex + 1];
      } else {
        // last step
        // console.log('last step');
      }
    },

    registrationStepEmailEntered(data) {
      this.confirmableAction = data.confirmableAction;
      this.registrationXid = data.registrationXid;
      this.sentToEmail = data.sentToEmail;
      this.gotoNext();
    },

    registrationStepInviteConfirmed() {
      this.gotoNext();
    },

    registrationStepInviteSkipped() {
      this.gotoNext();
    },

    registrationStepPhoneEntered(data) {
      this.confirmableAction = data.confirmableAction;
      this.registrationXid = data.registrationXid;
      this.sentToPhone = data.sentToPhone;
      this.gotoNext();
    },

    verifiedEmail() {
      // console.log('email verified');
      this.gotoNext();
    },

    verifiedPhone() {
      // console.log('phone verified');
      this.gotoNext();
    },

    changePhone() {
      this.registerStep = this.REGISTER_STEP_PHONE;
    },

    finishRegister() {
      this.registerStep = this.REGISTER_STEP_EMAIL_VERIFY;
      // window.location.replace('http://backend.bedrock.local' + this.registrationFinishedActionUrl)
      // console.log('finished');
    },
  },
};
</script>
