<template>
  <div>
    <div class="" :class="combinedStatus">
      <span v-if="isBaseLanguage">{{ $t('multilingual.misc.this_is_base_language') }}</span>

      <div class="d-flex justify-content-between">
        <div class="d-flex">
          <span class="status-icon" :class="this.text_status"></span>
          {{ textStatusIconText }}
          &nbsp;
          <span class="status-icon" :class="this.review_status"></span>
          {{ reviewStatusIconText }}
        </div>

        <div v-if="!isBaseLanguage">{{ meta.langBase }} => {{ state.toLang }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import store from '../Shared/store';

export default {
  data() {
    return {
      state: store.state,
    };
  },

  props: {
    meta: {
      type: Object,
      required: true,
    },
  },

  computed: {
    text_status() {
      if (this.meta.text_status === '') {
        return 'TRANSLATION_STATUS_TEXT_MISSING';
      }
      return this.meta.text_status;
    },

    review_status() {
      if (this.meta.review_status === '') {
        return 'TRANSLATION_STATUS_REVIEW_NEW';
      }
      return this.meta.review_status;
    },

    textStatusIconText() {
      return this.$t(`multilingual.status.textstatus.status.${this.text_status}.text`);
    },

    reviewStatusIconText() {
      return this.$t(`multilingual.status.reviewstatus.status.${this.review_status}.text`);
    },

    combinedStatus() {
      return this.meta.combinedStatusName;
    },

    isBaseLanguage() {
      return this.meta.langBase === this.meta.langTranslation;
    },
  },
};
</script>

<style scoped>
/*.updating::before {*/
/*    font-family: "Font Awesome 5 Free";*/
/*    font-weight: 900;*/
/*    content: "\f007";*/
/*}*/

/*.icon {*/
/*    display: block;*/
/*    text-color:red;*/
/*    text-indent: -9999px;*/
/*    width: 100px;*/
/*    height: 42px;*/
/*    background-size: 100px 42px;*/
/*}*/

/*.icon-new {*/
/*    !*background: url(/icons/gear.svg);*!*/
/*}*/

/*object {*/
/*    width: 10px;*/

/*}*/
/*.icci {*/

/*color:red;*/
/*    }*/

.TRANSLATION_STATUS_COMBINED_NEW {
  background-color: yellow !important;
}

.TRANSLATION_STATUS_COMBINED_MISSING {
  background-color: orange !important;
}

.TRANSLATION_STATUS_COMBINED_OUTDATED {
  background-color: orange !important;
}

.TRANSLATION_STATUS_COMBINED_BUSY {
  background-color: lime !important;
}
.TRANSLATION_STATUS_COMBINED_DONE {
  background-color: green !important;
}
.TRANSLATION_STATUS_COMBINED_DENIED {
  background-color: red !important;
}
.TRANSLATION_STATUS_COMBINED_UNDER_REVIEW {
  background-color: green !important;
}

img {
  height: 50px;
  vertical-align: middle;
}
</style>
