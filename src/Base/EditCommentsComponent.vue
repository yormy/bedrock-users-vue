<template>
  <v-app>
    <div class="d-flex-row">
      <v-textarea
        :name="comment"
        :id="comment"
        :label="$t('multilingual.comments.field') | capitalizeFirst"
        rows="2"
        filled
        hide-details="auto"
        auto-grow
        v-model="comment"
        :error-messages="validationErrors.comment"
      >
      </v-textarea>

      <div class="text-right">
        <button-submit :is-loading="isSubmitting" @clicked="addAction"> + </button-submit>
      </div>
    </div>

    <div v-if="isLoading">
      <v-chip class="ma-2" color="primary" outlined pill>{{ $t('multilingual.misc.loading') | capitalizeFirst }}...</v-chip>
    </div>

    <div v-else>
      <div v-if="!currentComments">No items found</div>
      <div v-else>
        <div v-for="item in currentComments" :key="item.id">
          <div class="d-flex">
            {{ item.comment }}
          </div>

          <div class="d-flex flex-column">
            <div class="text-right text-muted">
              <small>by {{ item.translator }}</small>
            </div>
            <div class="text-right text-muted">
              <small>On {{ item.created_at }}</small>
            </div>
          </div>
          <hr />
        </div>
      </div>

      <div class="mt-3">{{ $t('multilingual.comments.outdated') }}</div>

      <div v-for="item in outdatedComments" :key="item.id">
        <small class="d-flex text-muted">
          {{ item.comment }}
        </small>

        <div class="d-flex flex-column">
          <div class="text-right text-muted">
            <small>{{ $t('multilingual.misc.by_user') }} {{ item.translator }}</small>
          </div>
          <div class="text-right text-muted">
            <small>{{ $t('multilingual.misc.on_date') }} {{ item.created_at }}</small>
          </div>
        </div>
        <hr />
      </div>
    </div>
  </v-app>
</template>

<script>
import store from '../Shared/store';

export default {
  props: {
    indexRoute: {
      type: String,
      required: true,
    },

    addRoute: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      comment: '',
      currentComments: [],
      outdatedComments: [],
      isSubmitting: false,
      isLoading: false,
      validationErrors: [],
      state: store.state,
    };
  },

  mounted() {
    this.getComments();
  },

  watch: {
    'state.selectedItem': function () { /* eslint-disable-line */
      const self = this;
      setTimeout(() => {
        self.getComments();
      }, 0);
    },
  },

  methods: {
    addAction() {
      this.isSubmitting = true;

      const commentAddUrl = store.createUrl(this.route(this.addRoute, ['__ID__', '__I18N__']));

      const postdata = {
        comment: this.comment,
      };

      this.$http
        .post(commentAddUrl, postdata)
        .then((response) => {
          this.currentComments = response.data.data.currentComments;
          this.outdatedComments = response.data.data.outdatedComments;
          this.comment = '';
        })
        .catch((error) => {
          this.validationErrors = error.response.data.errors;
        })
        .finally(() => {
          this.isSubmitting = false;
        });
    },

    getComments() {
      if (!this.state.selectedItem || !this.state.selectedItem.meta.id) {
        this.currentComments = {};
        this.outdatedComments = {};
        return;
      }
      this.isLoading = true;

      const commentsUrl = store.createUrl(this.route(this.indexRoute, ['__ID__', '__I18N__']));

      this.$http
        .get(commentsUrl)
        .then((response) => {
          this.currentComments = response.data.data.currentComments;
          this.outdatedComments = response.data.data.outdatedComments;
        })
        .catch(() => {
          //
        })
        .finally(() => {
          this.isLoading = false;
        });
    },
  },
};
</script>
