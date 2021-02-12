<template>
  <div>
    <div v-if="!state.selectedItem">select an item on the left33</div>

    <transition name="fade">
      <div v-if="show">
        <div v-if="state.selectedItem">
<!--          <loading-overlay :show="isLoading"></loading-overlay>-->
          <v-card>
            <div v-if="!isAddItem()">
              <v-row>
                <v-col v-if="!isBaseLanguage" class="col-10">
                  <div class="small text-grey">
                    {{ state.selectedItem.staticFields.group }}.
                    {{ state.selectedItem.staticFields.key }}
                  </div>
                  "{{ state.selectedItem.fields.text.currentBase }}"
                </v-col>
                <v-col>
                  <div class="text-right">
                    <into-locale-switcher
                      :route-key="'user.multilingual.projects.tclabels.index'"
                      :project-id="projectId"
                      :to-lang="state.toLang"
                    >
                    </into-locale-switcher>
                  </div>
                </v-col>
              </v-row>

              <!--                <v-card-text class="pt-0">-->
              <!--                    <div v-if="!isBaseLanguage" class="d-flex pt-0">-->
              <!--                      full key : {{ state.selectedItem.staticFields.group }}.{{-->
              <!--                        state.selectedItem.staticFields.key-->
              <!--                      }}-->

              <!--                    </div>-->
              <!--                </v-card-text>-->

              <!--                <v-card-subtitle>-->

              <!--&lt;!&ndash;                  <p>&ndash;&gt;-->
              <!--&lt;!&ndash;                    ...TAGS...[show, add]&ndash;&gt;-->
              <!--&lt;!&ndash;                  </p>&ndash;&gt;-->
              <!--                </v-card-subtitle>-->
            </div>
            <div v-else>
              <v-card-title> NEW </v-card-title>
            </div>

            <v-card-text class="pt-0">
              <edit-header-component v-if="!isAddItem()" :meta="state.selectedItem.meta">
              </edit-header-component>

              <ValidationObserver ref="form">
                <div v-if="isBaseLanguage">
                  <div class="row">
                    <div class="col-md-12 pb-0">
                      <ValidationProvider v-slot="{ errors }" rules="required|min:2" name="group">
                        <v-text-field
                          name="group"
                          id="group"
                          :label="$t('multilingual.label.property.group') | capitalizeFirst"
                          v-model="state.selectedItem.staticFields.group"
                          hide-details="auto"
                          :error-messages="validationErrors.group ? validationErrors.group : errors"
                          :color="
                            !validationErrors.group && state.selectedItem.staticFields.group
                              ? 'success'
                              : ''
                          "
                          @keydown="validationErrors ? (validationErrors.group = '') : ''"
                          @keyup="updated()"
                        >
                        </v-text-field>
                      </ValidationProvider>
                    </div>
                  </div>

                  <div class="row">
                    <div class="col-md-12 pb-0">
                      <ValidationProvider v-slot="{ errors }" rules="required|min:2" name="key">
                        <v-text-field
                          name="key"
                          id="key"
                          :label="$t('multilingual.label.property.key') | capitalizeFirst"
                          v-model="state.selectedItem.staticFields.key"
                          hide-details="auto"
                          :error-messages="validationErrors.key ? validationErrors.key : errors"
                          :color="
                            !validationErrors.key && state.selectedItem.staticFields.key
                              ? 'success'
                              : ''
                          "
                          @keydown="validationErrors ? (validationErrors.key = '') : ''"
                          @keyup="updated()"
                        >
                        </v-text-field>
                      </ValidationProvider>
                    </div>
                  </div>
                </div>

                <div class="row">
                  <div class="col-md-12 pb-0">
                    <edit-input-component
                      :input-field="'text'"
                      :input-label="$t('multilingual.label.property.text') | capitalizeFirst"
                      :input-data.sync="state.selectedItem.fields.text.currentTranslated"
                      :api-errors="validationErrors.text"
                      validation-rules="required|min:5"
                      @updated="updated()"
                      :suggestion-allowed="suggestionAllowed()"
                    >
                    </edit-input-component>
                  </div>
                </div>

                <div class="form-group mt-12">
                  <edit-reviewer-component :review-status.sync="review_status" @updated="updated()">
                  </edit-reviewer-component>
                </div>
              </ValidationObserver>

              <div class="form-actions">
                <div class="card-body">
                  <div class="d-flex justify-content-between">
<!--                    <button-delete-->
<!--                      v-if="deleteAllowed() && !isAddItem()"-->
<!--                      :item="state.selectedItem"-->
<!--                      btn-class="btn btn-link text-danger"-->
<!--                      @deleteItem="deleteAgreed"-->
<!--                    >-->
<!--                      <template v-slot:delete-preview>-->
<!--                        {{ $t('multilingual.label.property.key') | capitalizeFirst}} :-->
<!--                        {{ state.selectedItem.staticFields.key }}-->
<!--                        <br />-->
<!--                        {{ $t('multilingual.label.property.group') | capitalizeFirst }}:-->
<!--                        {{ state.selectedItem.staticFields.group }}<br />-->
<!--                        {{ $t('multilingual.label.property.text') | capitalizeFirst}}: <br />-->
<!--                        {{ state.selectedItem.fields.text.currentTranslated }}-->
<!--                      </template>-->
<!--                    </button-delete>-->

                    <div v-if="isAddItem()">
<!--                      <button-submit-->
<!--                        id="btn-add"-->

<!--                        :is-loading="isLoading"-->
<!--                        @clicked="addAction"-->
<!--                      >-->
<!--                        {{ $t('multilingual.action.add') | capitalizeFirst }}-->
<!--                      </button-submit>-->
                      <button class="btn btn-primary" id="btn-test" @click="addAction">test</button>

                      <v-chip
                        class="ma-2"
                        close
                        color="red"
                        text-color="white"
                      >
                        Remove
                      </v-chip>

                    </div>
                    <div v-else>
<!--                      <button-submit-->
<!--                        id="btn-update"-->
<!--                        :disabled="!hasChanged"-->
<!--                        :is-loading="isLoading"-->
<!--                        @clicked="updateAction"-->
<!--                      >-->
<!--                        {{ $t('multilingual.action.update-and-next') | capitalizeFirst }}-->
<!--                      </button-submit>-->
                    </div>
                  </div>
                </div>
              </div>
            </v-card-text>
          </v-card>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import { ButtonDelete } from 'bedrock-vue-components';
import BaseEdit from '../Base/BaseEdit.vue';

export default {
  extends: BaseEdit,

  components: {
    ButtonDelete,
  },

  data() {
    return {
      defaultSuggestionField: 'text',
      translatables: ['text'],
      statics: ['key', 'group'],
    };
  },

  methods: {
    isValidForm() {
      this.$refs.form.validate();
      return this.$refs.form.flags.valid;
    },

    addAction() {
      console.error('form is valid?? ??');

      // if (!this.isValidForm()) {
      //   console.log('not valid form');
      //   //return;
      // }
      console.error('valid form');
      const fields = {
        group: this.state.selectedItem.staticFields.group,
        key: this.state.selectedItem.staticFields.key,
        text: this.state.selectedItem.fields.text.currentTranslated,
        review_status: this.review_status,
      };

      const addUrl = this.route('api.v1.user.multilingual.projects.tclabels.store', [
        this.projectId,
        this.state.toLang,
      ]);

      console.error(addUrl);
      this.doAddCall(fields, addUrl);
    },

    updateAction() {
      if (!this.isValidForm()) {
        return;
      }

      const fields = {
        group: this.state.selectedItem.staticFields.group,
        key: this.state.selectedItem.staticFields.key,
        text: this.state.selectedItem.fields.text.currentTranslated,
        review_status: this.review_status,
      };
      this.doUpdateCall(fields);
    },

    deleteAgreed() {
      this.doDeleteCall(this.state.selectedItem);
    },
  },
};
</script>

<style scoped>
.fade-enter-active {
  animation: slide-in 2s ease-out forwards;
  transition: opacity 1s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
</style>
