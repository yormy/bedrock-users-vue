<template>
  <v-card class="datatable">
    <div class="d-flex justify-content-between">
      <div>
        <div class="tabletitle">{{ title }}</div>
      </div>
      <div>
        <div class="tablesearch">
          <datatable-search :search-input.sync="searchInput"></datatable-search>
        </div>
      </div>
    </div>

    <v-data-table
      :headers="headers"
      :items="activities"
      :search="searchData"
      item-key="xid"
      single-select
      class="elevation-1"
      show-expand
    >
      <template v-slot:expanded-item="{ headers, item }">
        <td :colspan="headers.length">{{ item }} <br /></td>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
import { Datatable, DatatableSearch } from 'bedrock-vue-components';

export default {
  extends: Datatable,

  components: {
    DatatableSearch,
  },

  props: {
    title: {
      type: String,
    },

    withUser: {
      type: Boolean,
      default: false,
    },

    activities: {
      type: Array,
    },
  },

  data() {
    return {
      headers: null,
      searchInput: null,
    };
  },

  created() {
    this.createHeaders();
  },

  methods: {
    createHeaders() {
      this.headers = [];

      if (this.withUser) {
        this.headers.push(
          {
            text: this.$t('bedrock-users.misc.user'),
            value: 'person.xid',
          },
          {
            text: this.$t('bedrock-users.misc.email'),
            value: 'person.email',
          },
        );
      }

      this.headers.push(
        {
          text: this.$t('bedrock-users.misc.description'),
          value: 'description',
        },
        {
          text: this.$t('bedrock-users.misc.user_ip'),
          value: 'ip_user',
        },
        {
          text: this.$t('bedrock-users.misc.created_at_humans'),
          value: 'created_at_humans',
        },
        {
          text: 'd',
          value: 'dummy',
          class: 'hidden',
          width: '1px',
          align: ' d-none',
        } /* add dummy column to be able to additional default filtering on status */,
      );
    },
  },
};
</script>
