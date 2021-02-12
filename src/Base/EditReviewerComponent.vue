<template>
  <div>
    <v-select
      :label="$t('multilingual.misc.review_status.title')"
      :items="selectOptions"
      item-text="name"
      item-value="key"
      hide-details="auto"
      v-model="reviewStatusSelected"
      @change="inputChanged"
    >
    </v-select>
  </div>
</template>

<script>
export default {
  props: {
    reviewStatus: String,
  },

  data() {
    return {
      reviewStatusSelected: '',
      statuses: [
        'TRANSLATION_STATUS_REVIEW_NEW',
        'TRANSLATION_STATUS_REVIEW_AUTO',
        'TRANSLATION_STATUS_REVIEW_BUSY',
        'TRANSLATION_STATUS_REVIEW_REQUEST',
        'TRANSLATION_STATUS_REVIEW_DENIED',
        'TRANSLATION_STATUS_REVIEW_APPROVED',
      ],
      selectOptions: [],
    };
  },

  created() {
    this.statuses.forEach((value) => {
      this.selectOptions.push({
        name: this.$t(`multilingual.status.reviewstatus.status.${value}.text`),
        key: value,
      });
    });
  },

  mounted() {
    this.init();
  },

  watch: {
    reviewStatus() {
      this.init();
    },
  },

  methods: {
    updated() {
      this.$emit('updated');
    },

    init() {
      this.reviewStatusSelected = this.reviewStatus;
    },

    inputChanged() {
      this.$emit('update:reviewStatus', this.reviewStatusSelected);
      this.$emit('updated');
    },
  },
};
</script>

<style></style>
