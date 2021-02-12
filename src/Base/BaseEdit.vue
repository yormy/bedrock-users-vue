<script>
import EventBus from '../Shared/event-bus';
import store from '../Shared/store';
import EditHeaderComponent from './EditHeaderComponent.vue';
import EditInputComponent from './EditInputComponent.vue';
import EditReviewerComponent from './EditReviewerComponent.vue';
import IntoLocaleSwitcher from '../Shared/IntoLocaleSwitcher.vue';

export default {
  components: {
    EditHeaderComponent,
    EditInputComponent,
    EditReviewerComponent,
    IntoLocaleSwitcher,
  },

  props: {
    projectId: {
      type: String,
      required: true,
    },

    // addRoute: {
    //   type: String,
    //   required: true,
    // },

    updateRoute: {
      type: String,
      required: true,
    },

    deleteRoute: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      show: false,
      review_status: '',
      isLoading: false,
      hasChanged: false,
      validationErrors: [],
      deleteDialog: false,
      state: store.state,
    };
  },

  computed: {
    isBaseLanguage() {
      return this.state.selectedItem.meta.langBase === this.state.selectedItem.meta.langTranslation;
    },
  },

  watch: {
    'state.selectedItem': function () { /* eslint-disable-line */
      if (this.state.selectedItem) {
        this.selectedItemChanged();

        this.review_status = this.state.selectedItem.meta.review_status;

        this.hasChanged = false;
      }
    },

    'state.suggestion.value': function () { /* eslint-disable-line */
      this.updated();
    },

    // 'state.suggestion.value': function () { /* eslint-disable-line */
    //   if (this.state.suggestion.value) {
    //     const fieldName = this.state.suggestion.field;
    //     this.state.selectedItem.fields[fieldName].currentTranslated = this.selectedSuggestion;
    //     this.updated();
    //   }
    // },
  },

  mounted() {
    EventBus.$on('multilingual-item-new', this.createNewItem);
    EventBus.$on('multilingual-item-suggestion', this.suggestionSelected);
  },

  methods: {
    suggestionSelected() {
      const { field } = this.state.suggestion;
      this.state.selectedItem.fields[field].currentTranslated = this.state.suggestion.value;
    },

    createNewItem() {
      const newItem = {
        dummy: '@', // needed for searching and filtering
        meta: {
          id: 0,
          text_status: 'TRANSLATION_STATUS_TEXT_BASE_LANG',
          review_status: 'TRANSLATION_STATUS_REVIEW_NEW',
        },
        fields: {},
        staticFields: {},
      };

      this.translatables.forEach((field) => {
        newItem.fields[field] = { currentTranslated: '' };
      });

      this.statics.forEach((field) => {
        newItem.staticFields[field] = '';
      });

      this.state.selectedItem = newItem;
      this.state.selectedItemIndex = -1;
      window.scrollTo(0, 0);
    },

    updateListItemAdded(response) {
      const newItem = {
        dummy: '@', // needed for searching and filtering
        meta: {
          id: response.id,
          text_status: 'TRANSLATION_STATUS_TEXT_BASE_LANG',
          review_status: response.review_status[store.state.toLang],
        },
        fields: {},
        staticFields: {},
      };

      this.translatables.forEach((field) => {
        const translatedIntoCurrentLanguage = response[field][store.state.toLang];
        newItem.fields[field] = { currentTranslated: translatedIntoCurrentLanguage };
      });

      this.statics.forEach((field) => {
        newItem.staticFields[field] = response[field];
      });

      return newItem;
    },

    updateListItemUpdated(reviewStatus) {
      this.state.selectedItem.meta.text_status = 'TRANSLATION_STATUS_TEXT_OK'; // now text is ok
      this.state.selectedItem.meta.review_status = reviewStatus;

      // Update currentBase if the text has changes, as this is the language
      if (this.isBaseLanguage) {
        this.translatables.forEach((field) => {
          this.state.selectedItem.fields[field].currentBase = this.state.selectedItem.fields[
            field
          ].currentTranslated;
        });
      }
      return this.state.selectedItem;
    },

    selectedItemChanged() {
      this.show = false;

      window.setTimeout(() => {
        this.show = true;
      }, 0);
    },

    isAddItem() {
      if (
        !this.state.selectedItem.meta ||
        (this.state.selectedItem.meta && this.state.selectedItem.meta.id === 0)
      ) {
        return true;
      }
      return false;
    },

    updated() {
      this.hasChanged = true;
    },

    deleteAllowed() {
      return this.isBaseLanguage;
    },

    suggestionAllowed() {
      return !this.isBaseLanguage;
    },

    doAddCall(fields, addUrl) {
      this.isLoading = true;

      const data = {
        ...fields,
      };

      const self = this;
      this.$http
        .post(addUrl, data)
        .then((response) => {
          console.error(response)
          const itemData = response.data;
          const item = self.updateListItemAdded(itemData);

          store.addItem(item);
          self.itemResponseSuccess();
          EventBus.$emit('multilingual-item-added');
        })
        .catch((error) => {
          self.itemResponseFailed(error);
        });
    },

    doUpdateCall(fields) {
      this.isLoading = true;
      const updateUrl = store.createUrl(this.route(this.updateRoute, ['__ID__', '__I18N__']));

      const data = {
        ...fields,
      };

      const self = this;
      this.$http
        .put(updateUrl, data)
        .then(() => {
          const item = self.updateListItemUpdated(self.review_status);
          store.updateItem(item);
          store.selectNextItem();
          self.itemResponseSuccess();
          EventBus.$emit('multilingual-item-updated');
        })
        .catch((error) => {
          self.itemResponseFailed(error);
        });
    },

    doDeleteCall(item) {
      this.isLoading = true;
      const deleteUrl = store.createUrl(this.route(this.deleteRoute, ['__ID__', '__I18N__']));

      this.$http
        .delete(deleteUrl)
        .then(() => {
          store.deleteItem(item);
          this.itemResponseSuccess();
          EventBus.$emit('multilingual-item-deleted', item);
        })
        .catch((error) => {
          this.itemResponseFailed(error);
        });
    },

    itemResponseSuccess() {
      this.isLoading = false;
      this.hasChanged = false;
      this.deleteDialog = false;
      this.validationErrors = [];
      window.scrollTo(0, 0);
    },

    itemResponseFailed(error) {
      this.isLoading = false;
      this.hasChanged = false;
      this.deleteDialog = false;
      this.validationErrors = error.response.data.errors;
      window.scrollTo(0, 0);
    },
  },
};
</script>
