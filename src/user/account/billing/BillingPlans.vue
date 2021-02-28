<template>
  <div>

    <v-radio-group v-model="planPeriod">
      <v-radio
        value="YEARLY"
        :label="$t('bedrock-users.billing.plan.yearly')">
      </v-radio>
      <v-radio
        value="MONTHLY"
        :label="$t('bedrock-users.billing.plan.monthly')">
      </v-radio>
    </v-radio-group>

    <div class="row">

      <div v-for="plan in plans" :key="plan.xid" class="col-md-4 col-sm-6">
          <div class="pricingTable" :class="plan.color">
            <div class="pricingTable-header">
              <h3 class="preferred" v-if="plan.preferred">{{ $t('bedrock-users.billing.plan.preferred') }}</h3>
              <h3 class="title">{{ plan.name | capitalize }}</h3>
              <div class="price-value">
                <!-- <span class="currency">$</span>-->
                <span class="amount">{{ plan.price_formatted }}</span>
                <span class="duration">/{{ planPeriodText() }}</span>
              </div>
            </div>
            <ul class="pricing-content">
              <span v-html="plan.feature_list_html"></span>
            </ul>

            <div v-if="currentPlanId && plan.xid === currentPlanId">
              {{ $t('bedrock-users.billing.plan.your_current') }}
              <div v-if="onGracePeriod">
                <div class="d-flex flex-column">
                  <div>
                    {{ $t('bedrock-users.billing.plan.ends_at') }}
                    {{ planEndsAt }}
                  </div>
                  <div class="pricingTable-signup">
                    <a href="#"
                       :disabled="form.isLoading != null"
                       @click="subscribe(plan.xid)">
                      {{ $t('bedrock-users.billing.plan.resume') }}
                    </a>
                  </div>

                </div>
              </div>
              <div v-else>
                <button-submit
                  btnClass="btn btn-link"
                  :disabled="form.isLoading != null"
                  @clicked="cancel(plan.xid)"
                >
                  {{ $t('bedrock-users.action.cancel') }}
                </button-submit>
              </div>
            </div>
            <div v-else>
                <div class="pricingTable-signup">
                  <a href="#"
                     :disabled="form.isLoading != null"
                     @click="subscribe(plan.xid)">
                    {{ $t('bedrock-users.billing.plan.subscribe') }}
                  </a>
                </div>

            </div>

          </div>
        </div>


    </div>

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
        return this.$t('bedrock-users.billing.plan.year');
      }
      return this.$t('bedrock-users.billing.plan.month');
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
