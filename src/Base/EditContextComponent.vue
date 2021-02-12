<template>
  <v-app>
    <v-dialog v-model="dialogDelete" persistent max-width="500">
      <v-card>
        <v-card-title class="headline danger">
          {{ $t('actions.delete') }}
        </v-card-title>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" text @click="dialogDelete = false">{{
            $t('actions.cancel')
          }}</v-btn>
          <v-btn color="red darken-1" text @click="deleteAgreed">{{ $t('actions.delete') }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogDeleteOutdated" persistent max-width="500">
      <v-card>
        <v-card-title class="headline danger">
          {{ $t('actions.delete') }} ALL OUTDATED
        </v-card-title>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green darken-1" text @click="dialogDeleteOutdated = false">{{
            $t('actions.cancel')
          }}</v-btn>
          <v-btn color="red darken-1" text @click="deleteAgreedOutdated">{{
            $t('actions.delete')
          }}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-if="selectedScreenshot" v-model="dialog" :width="dialogWidth">
      <v-card>
        <v-card-title>
          <span class="headline">Screenshot {{ selectedScreenshot.url }}</span>
        </v-card-title>
        <v-card-text>
          <img :style="screenshotSize" :src="selectedScreenshot.screenshot" alt="screenshot" />
        </v-card-text>
      </v-card>
    </v-dialog>

    <div v-if="isLoading">
      <v-chip class="ma-2" color="primary" outlined pill> loading ... </v-chip>
    </div>

    <div v-else>
      <div v-if="!contexts.length">No items found</div>
      <div v-else>
        <div class="mb-2 text-right text-muted">
          <button @click="deleteOutdatedHandler()" class="btn btn-small btn-text text-danger">
            <span class="fas fa-trash-alt"></span>
            Outdated
          </button>
        </div>
        <hr />

        <div v-for="item in contexts" :key="item.id">
          <div v-if="item.status === 1"></div>
          <div v-if="item.status === 9">outdated</div>

          <div class="d-flex justify-content-center">
            <img
              style="border: 1px solid black"
              :style="thumbSize(item)"
              :src="item.screenshot"
              @click="screenshotClickHandler(item)"
              alt="screenshot"
            />
          </div>

          <div class="d-flex flex-column">
            <div class="text-right text-muted">
              <small>{{ item.url }}</small>
            </div>
            <div class="text-right text-muted">
              <span class="text-right text-muted">
                <button @click="deleteItemHandler(item)" class="btn btn-small btn-link text-danger">
                  <span class="fas fa-trash-alt"></span>
                </button>
              </span>

              <small>{{ item.created_at }}</small>
            </div>
          </div>
          <hr />
        </div>
      </div>
    </div>
  </v-app>
</template>

<script>
import store from '../Shared/store';

export default {
  props: {
    contextActionUrl: null,

    deleteActionUrl: {
      type: String,
      required: true,
    },

    deleteOutdatedActionUrl: {
      type: String,
      required: true,
    },

    selectedItem: null,
  },

  data() {
    return {
      dialog: false,
      dialogDelete: false,
      dialogDeleteOutdated: false,
      dialogScreenRatio: 0.95,
      deletingItem: null,
      tabWidth: 200,
      isLoading: false,
      selectedScreenshot: null,
      contexts: [],

      state: store.state,
    };
  },

  computed: {
    dialogWidth() {
      const dialogWidth = window.innerWidth * this.dialogScreenRatio;
      return `${dialogWidth.toString()}px`;
    },

    screenshotSize() {
      const dialogWidth = window.innerWidth * this.dialogScreenRatio;
      const widthRatio = dialogWidth / this.selectedScreenshot.screenshot_width;
      const widthRatioPercentage = widthRatio * 100;
      return `width:${widthRatioPercentage.toString()}%;height:auto`;
    },
  },

  mounted() {
    this.getContext();
  },

  watch: {
    selectedItem() {
      const self = this;
      setTimeout(() => {
        self.getContext();
      }, 0);
    },
  },

  methods: {
    deleteItemHandler(item) {
      this.dialogDelete = true;
      this.deletingItem = item;
    },

    deleteOutdatedHandler() {
      this.dialogDeleteOutdated = true;
    },

    deleteAgreed() {
      this.isLoading = true;
      const url = this.route(this.deleteActionUrl, [this.deletingItem.xid]);

      const self = this;
      this.$http
        .delete(url)
        .then(() => {
          self.getContext();
        })
        .catch((error) => {
          console.log(error); /* eslint-disable-line */
        })
        .finally(() => {
          self.dialogDelete = false;
        });
    },

    deleteAgreedOutdated() {
      this.isLoading = true;
      const url = this.route(this.deleteOutdatedActionUrl, [
        this.getCurrentKey(),
        this.state.toLang,
      ]);

      const self = this;
      this.$http
        .delete(url)
        .then(() => {
          self.getContext();
        })
        .catch(() => {})
        .finally(() => {
          self.dialogDeleteOutdated = false;
          // .bind(this)
        });
    },

    getCurrentKey() {
      let { group } = this.selectedItem.staticFields;
      group = group.replace('@', '::');
      const { key } = this.selectedItem.staticFields;
      return `${group}.${key}`;
    },

    thumbSize() {
      return 'width:100px;height:100px';
      // const widthRatio = this.tabWidth / item.screenshot_width;
      // const width = item.screenshot_width * widthRatio;
      // const height = item.screenshot_height * widthRatio;
      // return `width:${width.toString()}px;height:${height.toString()}px`;
    },

    screenshotClickHandler(item) {
      this.dialog = true;
      this.selectedScreenshot = item;
    },

    getContext() {
      this.isLoading = true;

      const url = this.route(this.contextActionUrl, [this.getCurrentKey(), this.state.toLang]);

      const self = this;
      this.$http
        .get(url)
        .then((response) => {
          self.contexts = response.data.data.contexts;
          self.isLoading = false;
        })
        .catch((error) => {
          self.validationErrors = error.response.data.errors;
          self.isLoading = false;
          self.hasChanged = false;
        });
    },
  },
};
</script>
