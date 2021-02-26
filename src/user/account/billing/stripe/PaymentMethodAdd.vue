<template>
  <div>
    <div class="card">
      <div class="card-header">{{ $t('billing.payment_method.add_method') }}</div>

      <div class="card-body">
        <ValidationObserver ref="form">
          <div class="row">
            <div class="col-md-4">
              <ValidationProvider v-slot="{ errors }" rules="required" name="name">
                <v-text-field
                  autocomplete=""
                  v-model="name"
                  :label="$t('billing.checkout.name_cardholder.label')"
                  :hint="$t('billing.checkout.name_cardholder.hint')"
                  :error-messages="apiErrors.name ? apiErrors.name : errors"
                  :color="!apiErrors.name && name ? 'success' : ''"
                  @keydown="apiErrors.name = ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>
          </div>
        </ValidationObserver>

        <div class="row">
          <div class="col-12">
            <div ref="cardAddPayment"></div>
            <div v-if="cardErrorMessage" class="text-danger">
              {{ $t('billing.checkout.card_error') }}: {{ cardErrorMessage }}
            </div>
          </div>
        </div>

        <hr />
        <v-checkbox
          v-model="asDefault"
          class="mt-0"
          :label="$t('billing.payment_method.mark_as_default')"
        >
        </v-checkbox>

        <button-submit
          :is-loading="state.isProccesingAdding"
          btnClass="btn btn-primary"
          @clicked="addPaymentMethod"
          id="btnBuy"
        >
          <strong>{{ $t('billing.payment_method.add_method') }}</strong>
        </button-submit>
      </div>
    </div>
  </div>
</template>

<script>
let card = null;

const style = {
  base: {
    fontSize: '16px',
  },

  invalid: {
    // All of the error styles go inside of here.
  },
};

export default {
  components: {},

  props: {
    clientSecret: {
      type: String,
    },
  },

  data: () => ({
    loading: false,

    stripe: null,
    elements: null,

    state: {
      isProccesingAdding: null,
    },

    name: null,
    asDefault: null,

    apiErrors: {},
    cardErrorMessage: null,
  }),

  created() {
    this.stripe = Stripe(process.env.VUE_APP_ENV_STRIPE_KEY); /* eslint-disable-line */
    this.elements = this.stripe.elements();
  },

  mounted() {
    // load standard stripe script
    // const stripeScript = document.createElement('script');
    // stripeScript.setAttribute('src', 'https://js.stripe.com/v3/');
    // document.head.appendChild(stripeScript);

    // mount stripe after vue elements to prevent conflicts
    // card = elements.create('card', style);

    if (card) {
      card.destroy(this.$refs.cardAddPayment);
    }
    card = this.elements.create('card', { style });
    card.mount(this.$refs.cardAddPayment);

    this.summary = this.totals;
  },

  methods: {
    addPaymentMethod() {
      this.state.isProccesingAdding = true;
      this.cardErrorMessage = null;

      let paymentMethod = null;
      if (paymentMethod) {
        return true;
      }

      this.stripe
        .confirmCardSetup(this.clientSecret, {
          payment_method: {
            card,
            billing_details: { name: this.name },
          },
        })
        .then(result => {
          if (result.error) {
            this.cardErrorMessage = result.error.message;
            this.state.isProccesingPayment = false;
          } else {
            paymentMethod = result.setupIntent.payment_method;
            this.submitMethodData(paymentMethod);
          }
        });
      return false;
    },

    submitMethodData(paymentMethod) {
      const data = {
        payment_method: paymentMethod,
        name: this.name,
        asDefault: this.asDefault,
      };

      const urlPaymentMethod = this.route('api.v1.user.account.billing.payment-method.store');
      this.$http
        .post(urlPaymentMethod, data)
        .then(() => {
          window.setTimeout(() => {
            window.location.reload();
          }, 0);
        })
        .catch(() => {})
        .finally(() => {
          // this.state.isProccesingAdding = false; let the reload reset the sate
        });
    },
  },
};
</script>

<style scoped>
.StripeElement {
  outline: 0;
  padding-top: 16px;
  height: 56px;
  border-color: rgb(158, 158, 158);
  border-style: solid;
  border-radius: 4px;
  border-width: 1px;
}
</style>
