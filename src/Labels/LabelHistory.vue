<template>
  <div>
    <v-app>
      <div v-if="isLoading">
        <v-chip class="ma-2" color="primary" outlined pill> loading ... </v-chip>
      </div>

      <div v-else>
        <div v-if="!history || !history.length">No item found</div>
        <div v-for="item in history" :key="item.id">
          <div v-html="item.text.diff"></div>

          <div class="d-flex flex-column">
            <div class="text-right text-muted">
              <small>By {{ item.user_name }}</small>
            </div>

            <div class="d-flex flex-row justify-content-between">
              <div class="text-muted">
                <small>{{ item.date }}</small>
              </div>
              <div class="d-flex text-muted">
                <span class="status-icon" :class="item.review_status"></span>
                <small>{{
                  $t(`multilingual.status.reviewstatus.status.${item.review_status}.text`)
                }}</small>
              </div>
            </div>
          </div>
          <hr />
        </div>
      </div>
    </v-app>
  </div>
</template>

<script>
import BaseHistory from '../Base/BaseHistory.vue';

export default {
  extends: BaseHistory,

  computed: {
    actionUrl() {
      let url = this.historyActionUrl.replace('__ID__', this.selectedItem.meta.id);
      url = url.replace('__I18N__', this.toLang);
      return url;
    },
  },
};
</script>
