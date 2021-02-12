<script>
import store from '../Shared/store';

export default {
  props: {
    data: null,
    projectId: {
      type: String,
      required: true,
    },

    selectedItemId: {
      type: String,
    },

    toLang: String,
    fromLang: String,
    allowCreate: {
      type: Boolean,
    },
  },

  data() {
    return {
      state: store.state,
    };
  },

  created() {
    this.state.items = this.data;
    this.state.toLang = this.toLang;
    this.state.fromLang = this.fromLang;
    this.state.allowCreate = this.allowCreate;
  },

  computed: {
    isBaseLang() {
      return this.state.fromLang !== this.state.toLang;
    },
  },

  methods: {
    rowClickHandler(item, index = 0) {
      this.showItemEdit(item, index);
    },

    showItemEdit(item, index = 0) {
      this.state.selectedItem = item;
      this.state.selectedItemIndex = index;
      window.scrollTo(0, 0);
    },
  },
};
</script>
