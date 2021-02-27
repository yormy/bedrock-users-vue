<template>
  <v-card>
    <v-card-title>
      <v-row>
        <v-col cols="6">
          <h1>{{ title }}</h1>
        </v-col>
        <v-col cols="6">
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
    >
      <template v-slot:[`item.actions`]="{ item }">
        <div v-if="item.status_delivered === 'SUPPRESSED'">
          {{ $t('bedrock-users.misc.email_suppressed') }}
        </div>
        <div v-else>
            <button type="button" class="btn btn-primary" @click="openEmail(item)">
            {{ $t('bedrock-users.action.view') }}
          </button>
        </div>
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

    openEmailUrl: {
      type: String,
      required: true,
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
    openEmail(item) {
      const url = this.openEmailUrl.replace('HASH', item.hash);
      window.open(url, '_blank');
    },

    createHeaders() {
      this.headers = [];

      if (this.withUser) {
        this.headers.push(
          {
            text: '#',
            value: 'person.xid',
          },
          {
            text: this.$t('bedrock-users.email'),
            value: 'person.email',
          },
        );
      }

      this.headers.push(
        {
          text: this.$t('bedrock-users.profile.email.recipient'),
          value: 'recipient_email',
        },
        {
          text: this.$t('bedrock-users.profile.email.subject'),
          value: 'subject',
        },
        {
          text: this.$t('bedrock-users.profile.email.opens'),
          value: 'opens',
        },
        {
          text: this.$t('bedrock-users.profile.email.clicks'),
          value: 'clicks',
        },
        {
          text: this.$t('bedrock-users.profile.email.sent_at'),
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
