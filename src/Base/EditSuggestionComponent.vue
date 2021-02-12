<template>
  <div>
    <v-dialog
      v-model="showExtensiveSuggestion"
      v-show="!isLoading && state.suggestion.field && isExtensiveSuggestion()"
    >
      <v-card>
        <slot name="modal">
          <v-card-title class="headline"> </v-card-title>

          <v-card-text>
            <v-card elevation="2">
              <div v-html="suggestion"></div>
            </v-card>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>

            <v-btn color="green darken-1" text @click="showExtensiveSuggestion = false">
              {{ $t('multilingual.action.cancel') }}
            </v-btn>

            <v-btn color="red darken-1" text @click="selectedHandler">
              {{ $t('multilingual.misc.use_suggestion') }}
            </v-btn>
          </v-card-actions>
        </slot>
      </v-card>
    </v-dialog>

    <v-card flat>
      <v-card-title>{{ $t('multilingual.misc.suggestion') | capitalizeFirst }}</v-card-title>
      <v-card-subtitle>google</v-card-subtitle>
      <v-card-text>
        <div v-if="isLoading">
          <v-chip class="ma-2" color="primary" outlined pill>
            {{ $t('multilingual.misc.loading') }}
          </v-chip>
        </div>
        <div v-if="!isLoading && state.suggestion.field && !isExtensiveSuggestion()">
          <button @click="selectedHandler">
            <v-chip class="ma-2" color="primary" outlined pill>
              {{ suggestion }}
            </v-chip>
          </button>
        </div>
      </v-card-text>

      <v-card-text> </v-card-text>
    </v-card>
  </div>
</template>

<script>
import EventBus from '../Shared/event-bus';
import store from '../Shared/store';

export default {
  props: {
    getRoute: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      isLoading: false,
      suggestion: '',
      state: store.state,
      showExtensiveSuggestion: false,
    };
  },

  mounted() {
    this.getSuggestions();

    EventBus.$on('multilingual-suggestion-requested', () => {
      this.state.suggestion.value = '';
      this.getSuggestions();
    });
  },

  watch: {
    'state.selectedItem': function () { /* eslint-disable-line */
      const self = this;
      setTimeout(() => {
        self.getSuggestions();
      }, 0);
    },
    //
    // 'state.suggestion.field': function () { /* eslint-disable-line */
    //   if (!this.state.suggestion.field) {
    //     this.state.suggestion.value = '';
    //   } else {
    //     const self = this;
    //     setTimeout(() => {
    //       self.getSuggestions();
    //     }, 0);
    //   }
    // },
  },

  methods: {
    selectedHandler() {
      this.state.suggestion.value = this.suggestion;
      this.showExtensiveSuggestion = false;
      EventBus.$emit('multilingual-item-suggestion');
    },

    skipSuggestions() {
      return false;
      // if ('TRANSLATION_STATUS_REVIEW_APPROVED' === this.selectedItem.meta.review_status) {
      //   return true
      // }
    },

    isExtensiveSuggestion() {
      return this.suggestion.length > 100;
    },

    getSuggestions() {
      if (!this.state.suggestion.field || this.skipSuggestions()) {
        this.isLoading = false;
        return;
      }

      this.isLoading = true;
      this.showExtensiveSuggestion = false;

      const suggestionUrl = store.createUrl(
        this.route(this.getRoute, ['__ID__', '__FIELD__', '__I18N__']),
      );

      const self = this;
      this.$http
        .get(suggestionUrl)
        .then((response) => {
          self.suggestion = response.data.data.suggestion;
          self.isLoading = false;

          if (self.isExtensiveSuggestion()) {
            this.showExtensiveSuggestion = true;
          }
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
