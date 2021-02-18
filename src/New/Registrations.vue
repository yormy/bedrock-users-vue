<template>
  <v-card>
    <v-card-title>
      <v-row class="pb-0">
        <v-col class="pb-0 pt-0">
          <h1>{{ title }}</h1>
        </v-col>
      </v-row>

      <v-row class="pb-0 pt-0">
        <v-col cols="6" class="pb-0 pt-0"> </v-col>
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
      <!--      <template v-slot:item.raw_content="{ item }">-->
      <!--        {{ item.raw_content | truncate(10) }}-->
      <!--      </template>-->
      <!--      <template v-slot:item.summary_changes="{ item }">-->
      <!--        {{ item.summary_changes | truncate(10) }}-->
      <!--      </template>-->

      <template v-slot:expanded-item="{ headers, item }">
        <td :colspan="headers.length">
          {{ item }}
        </td>
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
          text: this.$t('misc.email'),
          value: 'email',
        },
        {
          text: this.$t('misc.registration.locale'),
          value: 'locale',
        },
        {
          text: this.$t('misc.registration.timezone'),
          value: 'timezone',
        },
        {
          text: this.$t('misc.registration.terms_general'),
          value: 'terms_general_agreed',
        },
        {
          text: this.$t('misc.registration.terms_marketing'),
          value: 'terms_marketing_agreed',
        },
        {
          text: this.$t('misc.registration.invite_code'),
          value: 'invite_code',
        },
        {
          text: this.$t('misc.registration.is_finished'),
          value: 'is_finished',
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
