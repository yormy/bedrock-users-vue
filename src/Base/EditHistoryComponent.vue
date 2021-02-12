<template>
  <div class="card mb-0">
    <!--    <div v-if="showPreviousSource()" class="form-group">-->
    <!--      <div class="'div_translate_prev">-->
    <!--        <strong>{{ $t('multilingual.status.textversion.previousbase') }}</strong>-->
    <!--        <div>{{ model.previousBase }}</div>-->
    <!--      </div>-->
    <!--    </div>-->

    <div v-if="showPreviousSource()" class="form-group">
      <div class="div_translate_now">
        <strong>{{ $t('multilingual.status.textversion.differencebase') }}</strong>
        <div v-html="diffedBase"></div>
      </div>
    </div>

    <!--    <div class="form-group">-->
    <!--      <div class="'div_translate_now">-->
    <!--        <strong>{{ $t('multilingual.status.textversion.currentbase') }}</strong>-->
    <!--        <div v-html="model.currentBase"></div>-->
    <!--      </div>-->
    <!--    </div>-->
  </div>
</template>

<script>
import store from '../Shared/store';

export default {
  props: {
    field: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      state: store.state,
      diffedBase: store.state.selectedItem.fields[this.field].diffedBase,
    };
  },

  methods: {
    showPreviousSource() {
      return this.state.selectedItem.meta.text_status !== 'TRANSLATION_STATUS_TEXT_OK';
    },
  },
};
</script>

<style>
ins {
  background: lightgreen;
  text-decoration: none;
}

del {
  background: pink;
}
</style>
