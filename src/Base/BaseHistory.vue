<script>
import EventBus from '../Shared/event-bus';
import store from '../Shared/store';

export default {
  props: {
    historyRoute: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      history: null,
      isLoading: false,
      state: store.state,
    };
  },

  mounted() {
    this.getHistory();
    EventBus.$on('multilingual-item-updated', this.getHistory);
  },

  watch: {
    'state.selectedItem': function () { /* eslint-disable-line */
      const self = this;
      setTimeout(() => {
        self.getHistory();
      }, 0);
    },
  },

  methods: {
    getHistory() {
      if (!this.state.selectedItem || !this.state.selectedItem.meta.id) {
        this.history = {};
        return;
      }

      this.isLoading = true;
      const historyUrl = store.createUrl(this.route(this.historyRoute, ['__ID__', '__I18N__']));

      const data = {
        id: this.state.selectedItem.meta.id,
        lang: this.state.toLang,
      };

      const self = this;
      this.$http
        .get(historyUrl, data)
        .then((response) => {
          self.history = response.data.data.history;
          self.isLoading = false;
        })
        .catch(() => {
          self.isLoading = false;
        });
    },
  },
};
</script>

<style scoped>
/*!* Hard hide the column of the DUMMY data *!*/
/*::v-deep .v-data-table__wrapper thead tr th:nth-of-type(7) {*/
/*    display:none;*/
/*}*/
/*::v-deep .v-data-table__wrapper tr td:nth-of-type(7) {*/
/*    display:none;*/
/*}*/

::v-deep .v-data-table__wrapper thead tr th.hidden {
  display: none;
}

::v-deep .v-card__title.danger {
  background-color: red;
}

::v-deep tr.v-data-table__selected {
  background: #7d92f5 !important;
}

::v-deep tr.v-data-table__expanded__content {
  background: #c2c0c2 !important;
  border: none !important;
  box-shadow: none !important;
}
</style>
