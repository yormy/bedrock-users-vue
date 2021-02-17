<template>
  <v-card>
    <v-card-title>
      <v-row class="pb-0">
        <v-col class="pb-0 pt-0">
          <h1>{{ title }}</h1>
        </v-col>
      </v-row>

      <v-row>
        <v-col cols="6" class="pb-0 pt-0"> </v-col>
        <v-col cols="6" class="pb-0 pt-0">
          <datatable-search :search-input.sync="searchInput"></datatable-search>
        </v-col>
      </v-row>
    </v-card-title>
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
import { Datatable } from 'bedrock-vue-components';
import { DatatableSearch}  from 'bedrock-vue-components';

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
            text: this.$t('misc.user'),
            value: 'person.xid',
          },
          {
            text: this.$t('misc.email'),
            value: 'person.email',
          },
        );
      }

      this.headers.push(
        {
          text: this.$t('misc.description'),
          value: 'description',
        },
        {
          text: this.$t('misc.user_ip'),
          value: 'ip_user',
        },
        {
          text: this.$t('misc.created_at_humans'),
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
