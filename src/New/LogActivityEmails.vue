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
          {{ $t('bedrock-users.misc.open') | capitalizeFirst }}
        </button>
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
          text: this.$t('bedrock-core.dashboard.recent_emails.field.recipient'),
          value: 'recipient',
        },
        {
          text: this.$t('bedrock-core.dashboard.recent_emails.field.subject'),
          value: 'subject',
        },
        {
          text: this.$t('bedrock-core.dashboard.recent_emails.field.opens'),
          value: 'opens',
        },
        {
          text: this.$t('bedrock-core.dashboard.recent_emails.field.clicks'),
          value: 'clicks',
        },
        {
          text: this.$t('bedrock-users.misc.created_at'),
          value: 'created_at_humans',
        },
        {
          text: '',
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
