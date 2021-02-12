<template>
  <div>
    <div @keydown.esc="toggleFullscreen" tabindex="0">
      <div class="row">
        <div class="col-5" v-show="!fullScreen">
          <label-list :selected-item-id="selectedItemId"> </label-list>
        </div>

        <div :class="fullScreen ? 'col-10' : 'col-5'">
          <label-edit
            :project-id="projectId"
            update-route="api.v1.user.multilingual.tclabels.update"
            delete-route="api.v1.user.multilingual.tclabels.destroy"
          >
          </label-edit>
        </div>

        <div class="col-2">
          <v-card fixed>
            <v-card-title> tabs </v-card-title>
<!--            <v-tabs>-->
<!--              <v-tab v-if="isBaseLang" href="#suggestions"><v-icon>mdi-help</v-icon></v-tab>-->
<!--              <v-tab href="#history"><v-icon>mdi-history</v-icon></v-tab>-->
<!--              <v-tab href="#comments"><v-icon>mdi-chat-outline</v-icon></v-tab>-->

<!--              <v-tab-item :value="'suggestions'">-->
<!--                <edit-suggestion-component-->
<!--                  get-route="api.v1.user.multilingual.tclabels.suggestions"-->
<!--                >-->
<!--                </edit-suggestion-component>-->
<!--              </v-tab-item>-->

<!--              <v-tab-item :value="'history'">-->
<!--                <label-history-->
<!--                  history-route="api.v1.user.multilingual.tclabels.history"-->
<!--                ></label-history>-->
<!--              </v-tab-item>-->

<!--              <v-tab-item :value="'comments'">-->
<!--                <edit-comments-component-->
<!--                  index-route="api.v1.user.multilingual.tclabels.comments.index"-->
<!--                  add-route="api.v1.user.multilingual.tclabels.comments.add"-->
<!--                >-->
<!--                </edit-comments-component>-->
<!--              </v-tab-item>-->
<!--            </v-tabs>-->
          </v-card>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import BaseIndex from '../Base/BaseIndex.vue';
import EditSuggestionComponent from '../Base/EditSuggestionComponent.vue';
import EditCommentsComponent from '../Base/EditCommentsComponent.vue';

import labelList from './LabelList.vue';
import labelEdit from './LabelEdit.vue';
import labelHistory from './LabelHistory.vue';

export default {
  extends: BaseIndex,

  components: {
    labelList,
    labelEdit,
    labelHistory,
    EditSuggestionComponent,
    EditCommentsComponent,
  },

  data() {
    return {
      fullScreen: false,
    };
  },

  methods: {
    toggleFullscreen() {
      this.fullScreen = !this.fullScreen;
    },
  },
};
</script>

<style scoped>
::v-deep .v-tab {
  padding-left: 10px;
  padding-right: 10px;
  max-width: 10px;
  min-width: 30px;
}
</style>
