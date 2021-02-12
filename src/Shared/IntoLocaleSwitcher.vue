<template>
  <div>
    <ul class="navbar-nav">
      <li class="nav-item dropdown">
        <a
          class="nav-link dropdown-toggle pt-0 pb-0"
          href="#"
          id="navbarDropdown2"
          role="button"
          data-toggle="dropdown"
          aria-haspopup="true"
          aria-expanded="false"
        >
          <i class="flag-icon" :class="flagIcon"></i>
        </a>
        <div class="dropdown-menu dropdown-menu-right" aria-labelledby="navbarDropdown2">
          <a
            v-if="locale !== 'en'"
            class="dropdown-item"
            :href="
              route(routeKey, { project_xid: projectId, i18n: 'en', selected: getSelectedItem() })
            "
          >
            <i class="flag-icon flag-icon-us"></i> English</a
          >
          <a
            v-if="locale !== 'nl'"
            class="dropdown-item"
            :href="
              route(routeKey, { project_xid: projectId, i18n: 'nl', selected: getSelectedItem() })
            "
          >
            <i class="flag-icon flag-icon-nl"></i> Nederlands</a
          >
          <a
            v-if="locale !== 'de'"
            class="dropdown-item"
            :href="
              route(routeKey, { project_xid: projectId, i18n: 'de', selected: getSelectedItem() })
            "
          >
            <i class="flag-icon flag-icon-de"></i> Deutsch</a
          >
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import store from './store';

export default {
  props: {
    toLang: {
      type: String,
      required: true,
    },

    projectId: {
      type: String,
      required: true,
    },

    buttonTextDefault: {
      type: String,
      required: false,
    },

    routeKey: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      locale: '',
      state: store.state,
    };
  },

  computed: {
    flagIcon() {
      if (this.locale === 'en') {
        return 'flag-icon-us';
      }
      if (this.locale === 'nl') {
        return 'flag-icon-nl';
      }
      if (this.locale === 'de') {
        return 'flag-icon-de';
      }

      return 'flag-icon-us';
    },
  },

  mounted() {
    if (this.toLang) {
      this.locale = this.toLang;
    } else {
      this.locale = this.state.toLang;
    }
  },

  methods: {
    getSelectedItem() {
      if (this.state && this.state.selectedItem) {
        return this.state.selectedItem.meta.id;
      }

      return null;
    },
  },
};
</script>

<style scoped></style>
