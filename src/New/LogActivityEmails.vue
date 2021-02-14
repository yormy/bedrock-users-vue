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
      item-key="hash"
      single-select
      class="elevation-1"
      show-expand
    >
      <template v-slot:expanded-item="{ headers, item }">
        <td :colspan="headers.length">{{ item }} <br /></td>
      </template>

      <template v-slot:[`item.person.xid`]="{ item }">
        {{ item['person.xid'] | truncate(5) }}
      </template>

      <template v-slot:[`item.actions`]="{ item }">
        <button type="button" class="btn btn-primary" @click="openEmail(item)">
          {{ $t('misc.open') | capitalizeFirst }}
        </button>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
import Datatable from '@components/shared/Datatable/Datatable.vue';
import DatatableSearch from '@components/shared/Datatable/DatatableSearch.vue';

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

      openEmailUrl: this.route('admin.mails.sent.show'),
    };
  },

  created() {
    this.createHeaders();
  },

  methods: {
    openEmail(item) {
      const url = this.openEmailUrl.replace('HASH', item.hash);
      window.open(url, '_blank');
    },

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
          text: this.$t('misc.emails.recipient'),
          value: 'recipient',
        },
        {
          text: this.$t('misc.emails.subject'),
          value: 'subject',
        },
        {
          text: this.$t('misc.emails.opens'),
          value: 'opens',
        },
        {
          text: this.$t('misc.emails.clicks'),
          value: 'clicks',
        },
        {
          text: this.$t('misc.created_at_humans'),
          value: 'created_at_humans',
        },
        {
          text: this.$t('misc.actions'),
          value: 'actions',
          sortable: false,
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
