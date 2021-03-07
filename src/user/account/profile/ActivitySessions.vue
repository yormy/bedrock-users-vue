<template>
  <v-card>
    <v-card-title>
      <v-row>
        <v-col cols="6">
          <h1> {{ $t('bedrock-users.profile.session.title') }}</h1>
        </v-col>
        <v-col cols="6">
          <button-submit btn-class="btn btn-danger float-right" @clicked="removeOtherSessions()">
            {{ $t('bedrock-users.profile.session.delete_other_sessions') }}
          </button-submit>
        </v-col>
      </v-row>
    </v-card-title>

    <v-data-table
      :headers="headers"
      :items="table.values"
      :search="searchData"
      item-key="hash"
      single-select
      class="elevation-1"
    >
      <template v-slot:[`item.actions`]="{ item }">
        <div v-if="item.id === currentSession" type="button">
          {{ $t('bedrock-users.profile.session.current') }}
        </div>
        <button v-else type="button" class="btn btn-danger" @click="removeSession(item)">
          {{ $t('bedrock-users.actions.logout') }}
        </button>
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
      required: true,
    },

    currentSession: {
      type: String,
      required: true,
    },

    deleteSessionUrl: {
      type: String,
      required: true,
    },

    deleteAllOtherSessionsUrl: {
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
    removeSession(item) {
      this.backupTable.values = this.table.values;

      const foundIndex = this.findIndex(this.table.values, item, 'id');
      this.table.values.splice(foundIndex, 1);

      const url = this.deleteSessionUrl.replace('SESSIONID', item.id);

      this.deleteAction(url);
    },

    removeOtherSessions() {
      this.backupTable.values = this.table.values;

      // first collect indexes, then remove indexes from the end till the beginning
      // otherwise the indexes are no longer correct after deleting the first item
      const indexToDelete = [];
      this.table.values.forEach((item, index) => {
        if (item.id !== this.currentSession) {
          indexToDelete.push(index);
        }
      });

      indexToDelete.reverse();
      indexToDelete.forEach(index => {
        this.table.values.splice(index, 1);
      });

      const url = this.deleteAllOtherSessionsUrl;
      this.deleteAction(url);
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
          text: this.$t('bedrock-users.misc.ip'),
          value: 'ip_address',
        },
        {
          text: this.$t('bedrock-users.profile.session.user_agent'),
          value: 'user_agent',
        },
        {
          text: this.$t('bedrock-users.profile.session.last_activity'),
          value: 'last_activity',
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

    deleteAction(url) {
      this.startLoadingState();

      const payload = {
        current: this.currentSession,
      };

      this.$http
        .delete(url, { data: payload })
        .then(response => {
        })
        .catch(() => {
          this.table.values = this.backupTable.values;
        })
        .finally(() => {
          this.stopLoadingState();
        });
    },

    startLoadingState() {
      this.form.isSubmitting = true;
    },

    stopLoadingState() {
      this.form.isSubmitting = false;
    },
  },
};
</script>
