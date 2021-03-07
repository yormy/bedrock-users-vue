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
      :items="datatableValues"
      :search="searchData"
      item-key="xid"
      single-select
      class="elevation-1"
    >
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

    datatableValues: {
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

      this.headers.push(
        {
          text: this.$t('bedrock-users.misc.name'),
          value: 'name',
        },
        {
          text: this.$t('bedrock-users.misc.email'),
          value: 'email',
        },
        {
          text: this.$t('bedrock-users.misc.created_at'),
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
