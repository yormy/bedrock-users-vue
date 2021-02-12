<template>
  <v-select
    :items="[
      { name: $t('multilingual.status.reviewstatus.select.all'), key: '' },
      {
        name: $t('multilingual.status.reviewstatus.select.new'),
        key: 'TRANSLATION_STATUS_REVIEW_NEW',
      },
      {
        name: $t('multilingual.status.reviewstatus.select.auto'),
        key: 'TRANSLATION_STATUS_REVIEW_AUTO',
      },
      {
        name: $t('multilingual.status.reviewstatus.select.busy'),
        key: 'TRANSLATION_STATUS_REVIEW_BUSY',
      },
      {
        name: $t('multilingual.status.reviewstatus.select.request'),
        key: 'TRANSLATION_STATUS_REVIEW_REQUEST',
      },
      {
        name: $t('multilingual.status.reviewstatus.select.denied'),
        key: 'TRANSLATION_STATUS_REVIEW_DENIED',
      },
      {
        name: $t('multilingual.status.reviewstatus.select.approved'),
        key: 'TRANSLATION_STATUS_REVIEW_APPROVED',
      },
    ]"
    item-text="name"
    item-value="key"
    :label="$t('multilingual.action.select')"
    persistent-hint
    return-object
    single-line
    v-model="reviewStatusSelected"
  >
  </v-select>
</template>

<script>
export default {
  data() {
    return {
      reviewStatusSelected: '',
    };
  },

  watch: {
    reviewStatusSelected() {
      this.$emit('update:reviewStatusSelected', this.reviewStatusSelected);
    },
  },
};
</script>
