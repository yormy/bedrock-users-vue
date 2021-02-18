<template>
  <v-card>
    <v-card-title>
      <v-row class="pb-0">
        <v-col class="pb-0 pt-0">
          <h1>{{ title }}</h1>
        </v-col>
      </v-row>

      <v-row>
        <v-col cols="6" class="pb-0 pt-0"></v-col>
        <v-col cols="6" class="pb-0 pt-0">
          <datatable-search :search-input.sync="searchInput"></datatable-search>
        </v-col>
      </v-row>
    </v-card-title>

    <v-data-table
      :headers="headers"
      :items="datatableValues"
      :search="searchData"
      item-key="xid"
      single-select
      class="elevation-1"
      show-expand
    >
      <template v-slot:expanded-item="{ headers, item }">
        <td :colspan="headers.length">
          <span v-for="(value, field) in item.diff" :key="field">
            {{ field }}: <span v-html="value"></span><br
          /></span>
          <br />
          {{ item }} <br />
        </td>
      </template>

      <template v-slot:[`item.diff`]="{ headers, item }">
        <span v-for="(value, field) in item.diff" :key="field">
          <span v-if="'action' !== field"> {{ field }}:</span>
          <span v-html="value"></span>
          <br />
        </span>
      </template>

      <template v-slot:[`item.impersonator.email`]="{ item }">
        <span v-if="item.impersonator"><span class="fal fa-exclamation-triangle"></span></span>
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
          text: this.$t('misc.event'),
          value: 'event',
        },
        {
          text: this.$t('misc.ip-address'),
          value: 'ip_address',
        },
        {
          text: this.$t('misc.diff'),
          value: 'diff',
        },
        {
          text: this.$t('misc.impersonated'),
          value: 'impersonator.email',
        },
        {
          text: this.$t('misc.auditable'),
          value: 'auditable_type',
        },
        {
          text: this.$t('misc.ip-address'),
          value: 'ip_address',
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
