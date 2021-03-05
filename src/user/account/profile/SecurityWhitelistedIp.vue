<template>
  <v-card>
    <v-card-title>
      <v-row>
        <v-col>
          {{ $t('bedrock-users.profile.security.whitelisting.title')}}
        </v-col>
        <v-col>
          <div v-if="form.state.isSubmitting" class="text-right">
            <span class="fa fa-spinner fa-spin"></span>
          </div>
          <div v-else class="text-right">
            <div v-if="form.messages.success" class="text-success">
              {{ form.messages.success }}
            </div>
            <div v-if="form.messages.error" class="text-danger">
              {{ form.messages.error }}
            </div>
          </div>
        </v-col>
      </v-row>
    </v-card-title>

    <v-data-table
      :headers="headers"
      :items="table.values"
      :search="searchData"
      item-key="xid"
      single-select
      class="elevation-1"
    >

      <template v-slot:[`item.actions`]="{ item }">
        <button-delete :item="item" @deleteItem="deleteAgreed(item)">
          <template v-slot:delete-preview>{{ item.ip_address }}</template>
        </button-delete>
      </template>

    </v-data-table>
  </v-card>
</template>

<script>
import { Datatable } from 'bedrock-vue-components';

export default {
  extends: Datatable,

  props: {
    title: {
      type: String,
    },

    datatableValues: {
      type: Array,
    },

    urlWhitelistedIpDelete: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      headers: null,
      searchInput: null,

      table: {
        values: this.datatableValues,
      },

      form: {
        state: {
          isSubmitting: false,
        },

        messages: {
          success: '',
          error: '',
        },
      },
    };
  },

  created() {
    this.createHeaders();
  },

  methods: {
    createHeaders() {
      this.headers = [
        {
          text: this.$t('bedrock-users.misc.ip'),
          value: 'ip_address',
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
      ];
    },

    deleteAgreed(item) {
      this.table.values = this.deleteItemFromTable(this.table.values, item, 'xid');
      this.deleteModal = false;
      this.deleteIp(item);
    },

    deleteIp(item) {
      this.clearformResult();
      this.setSubmitting();

      const payload = {
        xid: item.xid,
      };

      this.$http
        .delete(this.urlWhitelistedIpDelete, { data: payload })
        .then(response => {
          if (response.data.success) {
            this.form.messages.success = response.data.message;
          } else {
            this.form.messages.error = response.data.message;
            this.table.values = this.restoreTable();
          }
        })
        .catch(error => {
          this.table.values = this.restoreTable();
          this.form.messages.error = error.response.data.message;
        })
        .finally(() => {
          this.clearSubmitting();
        });
    },

    setSubmitting() {
      this.form.state.isSubmitting = true;
    },

    clearSubmitting() {
      this.form.state.isSubmitting = false;
    },

    clearformResult() {
      this.form.messages.success = '';
      this.form.messages.error = '';
    },
  },
};
</script>
