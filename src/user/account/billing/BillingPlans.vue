<template>
  <div>
    <v-radio-group v-model="planPeriod">
      <v-radio label="Yearly" value="YEARLY"></v-radio>
      <v-radio label="Montly" value="MONTHLY"></v-radio>
    </v-radio-group>

    <v-row justify="space-between">
      <div v-for="plan in plans" :key="plan.xid">
        <v-col class="text-center">
          <div>
            <div v-if="plan.preferred">PREFERRED</div>
            <h3>{{ plan.name }}</h3>
            <b>{{ plan.price_formatted }} / {{ planPeriodText() }}</b>
            <hr />
            <!--            {{ route('checkout', $plan->id) }}-->

            <div v-if="currentPlanId && plan.xid === currentPlanId">
              your current plan

              <div v-if="onGracePeriod">
                end at
                {{ planEndsAt }}
                <button-submit
                  :is-loading="form.isLoading === plan.xid"
                  :disabled="form.isLoading != null"
                  @clicked="resume(plan.xid)"
                  btnClass="btn btn-secondary"
                >
                  ResUme
                </button-submit>
              </div>
              <div v-else>
                <button-submit
                  :is-loading="form.isLoading === plan.xid"
                  :disabled="form.isLoading != null"
                  @clicked="cancel(plan.xid)"
                  btnClass="btn btn-danger"
                >
                  CancEL
                </button-submit>
              </div>
            </div>
            <div v-else>
              <button-submit
                :is-loading="form.isLoading === plan.xid"
                :disabled="form.isLoading != null"
                @clicked="subscribe(plan.xid)"
              >
                Subscribe to {{ plan.name }}
              </button-submit>
            </div>
          </div>
        </v-col>
      </div>
    </v-row>
  </div>
</template>

<script>
export default {
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

      plans: this.monthlyPlans,
      planPeriod: 'MONTHLY',
    };
  },

  watch: {
    planPeriod() {
      if (this.planPeriod === 'YEARLY') {
        this.plans = this.yearlyPlans;
      } else {
        this.plans = this.monthlyPlans;
      }
    },
  },

  methods: {
    planPeriodText() {
      if (this.planPeriod === 'YEARLY') {
        return this.$t('billing.plans.yearly');
      }
      return this.$t('billing.plans.monthly');
    },

    subscribe(xid) {
      this.form.isLoading = xid;
      this.$emit('subscribe', xid);
    },

    cancel(xid) {
      this.form.isLoading = xid;
      this.$emit('cancel', xid);
    },

    resume(xid) {
      this.form.isLoading = xid;
      this.$emit('resume', xid);
    },
  },
};
</script>
