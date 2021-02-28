<template>
  <div>
    <div v-if="trialEndsAt" class="alert alert-info">
      {{ $t('bedrock-users.billing.subscription.trial_ends_at', {'date' : trialEndsAt} ) }}
    </div>

    <billing-plans
      v-if="form.showPlans"
      :monthly-plans="monthlyPlans"
      :yearly-plans="yearlyPlans"
      :current-plan-id="currentPlanId"
      :on-grace-period="onGracePeriod"
      :trial-ends-at="trialEndsAt"
      :plan-ends-at="planEndsAt"
      @subscribe="subscribe"
      @cancel="cancel"
      @resume="resume"
    >
    </billing-plans>

    <div v-else>
      <div v-if="!checkout.client_secret">
        <appearing-bullets :start-ms="0" :messages="messages"></appearing-bullets>
      </div>
    </div>

    <billing-checkout-stripe
      v-if="checkout.client_secret"
      :client-secret="checkout.client_secret"
      :plan="checkout.plan"
      :totals="checkout.totals"
      :countries="checkout.countries"
    >
    </billing-checkout-stripe>
  </div>
</template>

<script>
import BillingPlans from './BillingPlans.vue';
import BillingCheckoutStripe from './stripe/BillingCheckoutStripe.vue';
import { AppearingBullets } from 'bedrock-vue-components';

export default {
  components: {
    BillingCheckoutStripe,
    BillingPlans,
    AppearingBullets
  },

  props: {
    monthlyPlans: {
      type: Array,
      required: true,
    },

    yearlyPlans: {
      type: Array,
      required: true,
    },

    currentPlanId: {
      type: String,
      default: null,
    },

    onGracePeriod: {
      default: null,
    },

    planEndsAt: {
      type: String,
      default: null,
    },

    trialEndsAt: {
      type: String,
      default: null,
    },
  },

  data() {
    return {
      form: {
        isLoading: null,
        showPlans : true,
      },

      checkout: {
        plan: null,
        totals: null,
        countries: null,
        client_secret: null,
      },

      paymentMethod: {
        client_secret: null,
      },

      messages: [
        {
          text: this.$t('bedrock-users.billing.checkout.waiting_messages.message1'),
          loading: false,
          completed: false,
          processingMs: 1000,
        },
        {
          text: this.$t('bedrock-users.billing.checkout.waiting_messages.message2'),
          loading: false,
          completed: false,
          processingMs: 2000,
        },
        {
          text: this.$t('bedrock-users.billing.checkout.waiting_messages.message3'),
          loading: false,
          completed: false,
          processingMs: 2000,
        },
      ],
    };
  },

  methods: {
    subscribe(xid) {
      this.form.isLoading = xid;
      this.form.showPlans = false;

      this.checkout.client_secret = null;
      const url = this.route('api.v1.user.account.billing.checkout.stripe.plan', xid);

      this.$http
        .get(url)
        .then(response => {
          if (response.data.success) {
            // this.state.successMessage = response.data.data;

            const resultData = response.data.data;
            if (resultData.redirect_to) {
              window.location.href = resultData.redirect_to;
            }
            this.checkout.plan = resultData.plan;
            this.checkout.client_secret = resultData.client_secret;
            this.checkout.countries = resultData.countries;
            this.checkout.totals = resultData.totals;
          }
        })
        .catch(() => {
          this.form.showPlans = true;
        })
        .finally(() => {
          this.form.isLoading = null;
        });
    },

    cancel(xid) {
      this.form.isLoading = xid;
      const url = this.route('api.v1.user.account.billing.cancel');

      this.$http
        .get(url)
        .then(response => {
          if (response.data.success) {
            const resultData = response.data.data;
            if (resultData.redirect_to) {
              window.location.href = resultData.redirect_to;
            }
          }
        })
        .catch(() => {})
        .finally(() => {
          // this.state.isSubmitting = false;
          this.form.isLoading = null;
        });
    },

    resume(xid) {
      this.form.isLoading = xid;
      const url = this.route('api.v1.user.account.billing.resume');

      this.$http
        .get(url)
        .then(response => {
          if (response.data.success) {
            const resultData = response.data.data;
            if (resultData.redirect_to) {
              window.location.href = resultData.redirect_to;
            }
          }
        })
        .catch(() => {})
        .finally(() => {
          // this.state.isSubmitting = false;
          this.form.isLoading = null;
        });
    },
  },
};
</script>
