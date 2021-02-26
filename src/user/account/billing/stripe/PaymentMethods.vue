<template>
  <div>
    <loading-overlay :show="form.isSubmitting"></loading-overlay>

    <payment-method-add v-if="clientSecret" :client-secret="clientSecret"> </payment-method-add>

    <table class="table">
      <thead>
        <tr>
          <td>{{ $t('billing.payment_method.brand') }}</td>
          <td>{{ $t('billing.payment_method.expiry') }}</td>
          <td>{{ $t('billing.payment_method.last') }}</td>
          <td></td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="paymentMethod in paymentMethods" :key="paymentMethod.id">
          <td>{{ paymentMethod.brand }}</td>
          <td>{{ paymentMethod.exp_month }} / {{ paymentMethod.exp_year }}</td>
          <td>{{ paymentMethod.last4 }}</td>
          <td>
            <div v-if="defaultPaymentMethodId === paymentMethod.id">default</div>
            <div v-else>
              <!--              :is-loading="form.isLoading === plan.xid"-->
              <!--              :disabled="form.isLoading != null"-->
              <button-submit @clicked="markAsDefault(paymentMethod.id)" btnClass="btn btn-link">
                {{ $t('billing.payment_method.mark_as_default') }}
              </button-submit>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import PaymentMethodAdd from './PaymentMethodAdd.vue';

export default {
  components: {
    PaymentMethodAdd,
  },

  props: {
    title: {
      type: String,
    },

    paymentMethods: {
      type: Array,
      required: true,
    },

    defaultPaymentMethodId: {
      type: String,
      default: null,
    },

    clientSecret: {
      type: String,
      default: null,
    },
  },

  data() {
    return {
      form: {
        isSubmitting: false,
      },
    };
  },

  methods: {
    markAsDefault(paymentMethodId) {
      this.form.isSubmitting = true;

      // this.checkout.client_secret = null;
      const url = this.route(
        'api.v1.user.account.billing.payment-method.markAsDefault',
        paymentMethodId,
      );

      this.$http
        .get(url)
        .then(response => {
          if (response.data.success) {
            window.setTimeout(() => {
              window.location.reload();
            }, 0);
          }
        })
        .catch(() => {})
        .finally(() => {
          this.form.isSubmitting = false;
        });
    },
  },
};
</script>
