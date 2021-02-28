<template>
  <div>
    <div>{{ $t('bedrock-users.billing.title') }}</div>
    <div v-if="trialEndsAt" class="alert alert-info">
      {{ $t('bedrock-users.billing.subscription.trial_ends_at', {'date' : trialEndsAt} ) }}
    </div>

    <billing-plans
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

export default {
  components: {
    BillingCheckoutStripe,
    BillingPlans,
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
    };
  },

  methods: {
    subscribe(xid) {
      this.form.isLoading = xid;

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
        .catch(() => {})
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
