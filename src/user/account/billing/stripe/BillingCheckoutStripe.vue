<template>
  <div>
    <div class="card">
      <div class="card-header">Subscribe to {{ plan.name }}</div>
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

            <div class="col-md-4">
              <ValidationProvider v-slot="{ errors }" rules="" name="address_line_1">
                <v-text-field
                  autocomplete=""
                  v-model="address_line_1"
                  :label="$t('billing.checkout.address_line_1.label')"
                  :hint="$t('billing.checkout.address_line_1.hint')"
                  :error-messages="apiErrors.address_line_1 ? apiErrors.address_line_1 : errors"
                  :color="!apiErrors.address_line_1 && address_line_1 ? 'success' : ''"
                  @keydown="apiErrors.address_line_1 = ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>

            <div class="col-md-4">
              <ValidationProvider v-slot="{ errors }" rules="" name="address_line_2">
                <v-text-field
                  autocomplete=""
                  v-model="address_line_2"
                  :label="$t('billing.checkout.address_line_2.label')"
                  :hint="$t('billing.checkout.address_line_2.hint')"
                  :error-messages="apiErrors.address_line_2 ? apiErrors.address_line_2 : errors"
                  :color="!apiErrors.address_line_2 && address_line_2 ? 'success' : ''"
                  @keydown="apiErrors.address_line_2 = ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>
          </div>

          <div class="row">
            <div class="col-md-4">
              <ValidationProvider v-slot="{ errors }" rules="" name="country">
                <v-autocomplete
                  :items="countries"
                  item-text="name"
                  item-value="iso3"
                  :label="$t('billing.checkout.country.label')"
                  :hint="$t('billing.checkout.country.hint')"
                  v-model="country"
                  :error-messages="apiErrors.country ? apiErrors.country : errors"
                  :color="!apiErrors.country && country ? 'success' : ''"
                  @keydown="apiErrors.country = ''"
                  outlined
                ></v-autocomplete>
              </ValidationProvider>
            </div>

            <div class="col-md-4">
              <ValidationProvider v-slot="{ errors }" rules="" name="city">
                <v-text-field
                  autocomplete=""
                  v-model="city"
                  :label="$t('billing.checkout.city.label')"
                  :hint="$t('billing.checkout.city.hint')"
                  :error-messages="apiErrors.city ? apiErrors.city : errors"
                  :color="!apiErrors.city && city ? 'success' : ''"
                  @keydown="apiErrors.city = ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>

            <div class="col-md-4">
              <ValidationProvider v-slot="{ errors }" rules="" name="zipcode">
                <v-text-field
                  autocomplete=""
                  v-model="zipcode"
                  :label="$t('billing.checkout.zipcode.label')"
                  :hint="$t('billing.checkout.zipcode.hint')"
                  :error-messages="apiErrors.zipcode ? apiErrors.zipcode : errors"
                  :color="!apiErrors.zipcode && zipcode ? 'success' : ''"
                  @keydown="apiErrors.zipcode = ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
            </div>
          </div>

          <div class="row">
            <div class="col-md-4">
              <ValidationProvider v-slot="{ errors }" rules="" name="coupon_code">
                <v-text-field
                  autocomplete=""
                  v-model="coupon_code"
                  :label="$t('billing.checkout.coupon_code.label')"
                  :hint="$t('billing.checkout.coupon_code.hint')"
                  :append-icon="coupon_valid ? 'fal fa-check' : ''"
                  :error-messages="apiErrors.coupon_code ? apiErrors.coupon_code : errors"
                  :color="!apiErrors.coupon_code && coupon_code ? 'success' : ''"
                  @keydown="apiErrors.coupon_code = ''"
                  outlined
                >
                </v-text-field>
              </ValidationProvider>
              <div v-if="coupon_text_valid" class="text-success">{{ coupon_text_valid }}</div>
              <div v-if="coupon_text_error" class="text-danger">{{ coupon_text_error }}</div>
            </div>

            <div class="col-md-8">
              <button-submit :is-loading="state.isApplyingCoupon" @clicked="applyCoupon">
                <strong>{{ $t('billing.checkout.apply_coupon') | capitalizeFirst }}</strong>
              </button-submit>
            </div>
          </div>
        </ValidationObserver>

        <div class="row">
          <div class="col-12">
            <div ref="card"></div>
            <div v-if="cardErrorMessage" class="text-danger">
              {{ $t('billing.checkout.card_error') }}: {{ cardErrorMessage }}
            </div>
          </div>
        </div>

        <div class="row">
          <div class="col-md-4">
            <div class="row">
              <div class="col-6 py-0">
                {{ $t('billing.checkout.summary.price.label') }}
              </div>
              <div class="col-6 py-0 text-right">
                <span class="" id="summary_price">
                  {{ plan.price_in_cents | fromCents() | currency(plan.currency_symbol) }}
                </span>
              </div>
            </div>

            <div v-if="summary.discountAmount" class="row">
              <div class="col-6 py-0">
                {{ $t('billing.checkout.summary.discount.label') }}
              </div>
              <div class="col-6 py-0 text-right">
                <span class="" id="summary_discountamount">
                  {{ summary.discountAmount | fromCents() | currency(plan.currency_symbol) }}
                </span>
              </div>
            </div>

            <div class="row">
              <div class="col-6 py-0">
                {{ $t('billing.checkout.summary.subtotal.label') }}
              </div>
              <div class="col-6 pt-0 text-right">
                <span class="font-weight-bold" id="summary_subtotal">
                  {{ summary.subtotal | fromCents() | currency(plan.currency_symbol) }}
                </span>
              </div>
            </div>

            <div class="row">
              <div class="col-6 py-0">
                {{ $t('billing.checkout.summary.tax.label', { tax: summary.taxPercent }) }}
              </div>
              <div class="col-6 py-0 text-right">
                <span class="" id="summary_taxamount">
                  {{ summary.taxAmount | fromCents() | currency(plan.currency_symbol) }}
                </span>
              </div>
            </div>

            <div class="row">
              <div class="col-6 py-0">
                {{ $t('billing.checkout.summary.total.label') }}
              </div>
              <div class="col-6 py-0 text-right">
                <span class="font-weight-bold" id="summary_total">
                  {{ summary.total | fromCents() | currency(plan.currency_symbol) }}
                </span>
              </div>
            </div>
          </div>
        </div>

        <hr />

        <button-submit
          :is-loading="state.isProccesingPayment"
          :disabled="state.isApplyingCoupon"
          btnClass="btn btn-primary"
          @clicked="buynow"
          id="btnBuy"
        >
          <strong>Pay {{ summary.total | fromCents() | currency(plan.currency_symbol) }}</strong>
        </button-submit>
      </div>
    </div>
  </div>
</template>

<script>
import { mergeErrors } from 'bedrock-vue-components';

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
      default: null,
    },

    plan: {
      type: Object,
      required: true,
    },

    totals: {
      type: Object,
      required: true,
    },

    countries: {
      type: Array,
      required: true,
    },
  },

  data: () => ({
    loading: false,

    stripe: null,
    elements: null,

    state: {
      isApplyingCoupon: null,
      isProccesingPayment: null,
      clientIntent: null,
    },

    coupon_discount_percent: null,
    coupon_discount_amount: null,

    name: null,
    address_line_1: null,
    address_line_2: null,
    country: null,
    city: null,
    zipcode: null,
    coupon_code: null,
    coupon_text_valid: null,
    coupon_text_error: null,
    coupon_valid: false,

    summary: {},

    apiErrors: {},
    cardErrorMessage: null,
  }),

  watch: {
    plan() {
      this.resetCouponCalculation();
      this.summary = this.totals;
    },
  },

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
      card.destroy(this.$refs.card);
    }
    card = this.elements.create('card', { style });
    card.mount(this.$refs.card);

    this.summary = this.totals;

    this.state.clientIntent = this.clientSecret;
  },

  methods: {
    buynow() {
      this.state.isProccesingPayment = true;
      this.cardErrorMessage = null;

      let paymentMethod = null;
      if (paymentMethod) {
        return true;
      }

      this.stripe
        .confirmCardSetup(this.state.clientIntent, {
          payment_method: {
            card,
            billing_details: {
              name: this.name,
              // email
              address: {
                line1: this.address_line_1,
                line2: this.address_line_2,
                city: this.city,
                country: this.country,
              },
            },
          },
        })
        .then(result => {
          if (result.error) {
            this.cardErrorMessage = result.error.message;
            this.state.isProccesingPayment = false;
          } else {
            paymentMethod = result.setupIntent.payment_method;
            this.submitPaymentData(paymentMethod);
          }
        });
      return false;
    },

    submitPaymentData(paymentMethod) {
      const data = {
        payment_method: paymentMethod,
        billing_plan_xid: this.plan.xid,
        name: this.name,
        address_line_1: this.address_line_1,
        address_line_2: this.address_line_2,
        country: this.country,
        city: this.city,
        zipcode: this.zipcode,
        coupon_code: this.coupon_code,
      };

      const urlPayment = this.route('api.v1.user.account.billing.checkout.stripe.process');
      this.$http
        .post(urlPayment, data)
        .then(response => {
          const resultData = response.data.data;
          // do not enable pay button, reload is still processing
          if (resultData.redirect_to) {
            window.location.href = resultData.redirect_to;
          }
        })
        .catch(error => {
          this.apiErrors = mergeErrors(error);
          this.apiErrors = error.response.data.data;
          this.state.clientIntent = error.response.data.data.new_client_intent;
          this.state.isProccesingPayment = false;
        })
        .finally(() => {});
    },

    resetCouponCalculation() {
      this.coupon_valid = false;
      this.coupon_text_valid = '';
      this.coupon_text_error = '';
      this.coupon_discount_percent = 0;
      this.coupon_discount_amount = 0;
      this.summary.discountAmount = 0;
      this.recalculateTotals();
    },

    applyCoupon() {
      this.state.isApplyingCoupon = true;

      const urlCoupon = this.route('api.v1.user.account.billing.checkout.coupon', {
        coupon: this.coupon_code,
      });

      this.resetCouponCalculation();

      this.$http
        .get(urlCoupon)
        .then(response => {
          const responseData = response.data.data;
          this.coupon_text_valid = responseData.description;
          this.coupon_discount_percent = responseData.percent_off;
          this.coupon_discount_amount = responseData.amount_off;
          this.coupon_valid = true;

          const amountOff = response.data.data.amount_off;
          if (amountOff) {
            this.summary.discountAmount = amountOff;
          } else {
            this.summary.discountAmount =
              this.plan.price_in_cents * (this.coupon_discount_percent / 100);
          }

          this.recalculateTotals();
        })
        .catch(error => {
          this.coupon_text_error = error.response.data.message;
        })
        .finally(() => {
          this.state.isApplyingCoupon = false;
        });
    },

    recalculateTotals() {
      this.summary.subtotal = this.plan.price_in_cents - this.summary.discountAmount;
      this.summary.taxAmount = this.summary.subtotal * (this.summary.taxPercent / 100);
      this.summary.total = this.summary.subtotal + this.summary.taxAmount;
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
