<template>
  <v-select
    :items="[
      { name: $t('multilingual.status.textstatus.select.all'), key: '' },
      {
        name: $t('multilingual.status.textstatus.select.missing'),
        key: 'TRANSLATION_STATUS_TEXT_MISSING',
      },
      {
        name: $t('multilingual.status.textstatus.select.old'),
        key: 'TRANSLATION_STATUS_TEXT_OUTDATED',
      },
      {
        name: $t('multilingual.status.textstatus.select.ok'),
        key: 'TRANSLATION_STATUS_TEXT_OK',
      },
    ]"
    item-text="name"
    item-value="key"
    :label="$t('multilingual.action.select')"
    persistent-hint
    return-object
    single-line
    v-model="textStatusSelected"
  ></v-select>
</template>

<script>
export default {
  data() {
    return {
      textStatusSelected: '',
    };
  },

  watch: {
    textStatusSelected() {
      this.$emit('update:textStatusSelected', this.textStatusSelected);
    },
  },
};
</script>
