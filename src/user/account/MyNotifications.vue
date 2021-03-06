<template>
  <div>
    <v-app>
      <div id="labels_overview_table">
        <v-card-title>
          <!--            {{$t('multilingual-admin.label.title')}}-->
          <v-row>
            <v-col cols="4">
              <list-search-component :search-input.sync="searchInput"></list-search-component>
            </v-col>
            <v-spacer></v-spacer>
            <v-col cols="4">
              <button type="button" class="btn btn-secondary float-right" @click="markAllAsRead">
                {{ $t('bedrock-users.misc.notification.mark_all_read') }}
              </button>
            </v-col>
          </v-row>
        </v-card-title>

        <v-data-table
          :headers="headers"
          :items="data"
          :search="searchData"
          :custom-filter="filter"
          :items-per-page="10"
          @click:row="rowClick"
          @current-items="getFiltered"
          item-key="id"
          single-select
          show-expand
          class="elevation-1"
          @item-expanded="expanded"
        >
          v-model="selectedItems" >
          <template v-slot:[`item.is_read`]="{ item }">
            <span v-if="!item.is_read">
              <span class="text-danger"><span class="fal fa-star-exclamation"></span></span>
            </span>
            <span v-else></span>
          </template>

          <template v-slot:[`item.summary`]="{ item }">
            {{ item.summary | truncate(20) }}
          </template>

          <template v-slot:expanded-item="{ headers, item }">
            <td :colspan="headers.length">
              {{ item.summary }}
              <div v-if="item.is_read">
                <br />
                <div class="text-right">
                  <button type="button" class="btn btn-sm btn-link" @click="markAsUnread(item)">
                    {{ $t('bedrock-users.misc.notification.mark_as_unread') }}
                  </button>
                </div>
              </div>
            </td>
          </template>
        </v-data-table>
      </div>
    </v-app>
  </div>
</template>

<script>
import { BaseList, ListSearchComponent} from 'bedrock-vue-components';

export default {
  extends: BaseList,

  components: {
    ListSearchComponent,
  },

  props: {
    title: {
      type: String,
    },

    data: {
      type: Array,
      required: true,
    },
  },

  methods: {
    expanded(item) {
      this.markAsRead(item.item);
    },

    setStateRead(item, read) {
      const index = this.findIndexBeta(item.id, 'id');
      this.filteredList[index].is_read = read;
    },

    setStateAllRead() {
      this.filteredList.forEach((item, index) => {
        this.filteredList[index].is_read = true;
      });
    },

    markAsRead(item) {
      if (item.is_read) {
        return;
      }

      const url = this.route('api.v1.member.account.notifications.mark-read', item.id);

      this.$http
        .put(url, {})
        .then(() => {
          this.setStateRead(item, true);
        })
        .catch(() => {})
        .finally(() => {});
    },

    markAsUnread(item) {
      if (!item.is_read) {
        return;
      }

      const url = this.route('api.v1.member.account.notifications.mark-unread', item.id);

      this.$http
        .put(url, {})
        .then(() => {
          this.setStateRead(item, false);
        })
        .catch(() => {})
        .finally(() => {});
    },

    markAllAsRead() {
      const url = this.route('api.v1.member.account.notifications.mark-all-read');

      this.$http
        .put(url, {})
        .then(() => {
          this.setStateAllRead();
        })
        .catch(() => {})
        .finally(() => {});
    },

    setHeaders() {
      this.headers = [
        {
          text: '',
          value: 'is_read',
        },
        {
          text: this.$t('bedrock-users.misc.subject'),
          value: 'subject',
        },
        {
          text: this.$t('bedrock-users.misc.summary'),
          value: 'summary',
        },
        {
          text: this.$t('bedrock-users.misc.sent_at'),
          value: 'created_at',
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
  },
};
</script>
