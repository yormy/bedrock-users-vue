<template>
  <div>
    <div class="card">
      <v-app>
        <div id="labels_overview_table">
          <v-card-title>
            <v-row>
              <v-col>
                {{ $t('multilingual.project.overview.title') }}
              </v-col>
              <v-spacer></v-spacer>
              <v-col class="text-right">
                <into-locale-switcher
                  :route-key="'user.multilingual.projects.show'"
                  :project-id="project.xid"
                  :to-lang="toLang"
                >
                </into-locale-switcher>
              </v-col>
            </v-row>
          </v-card-title>

          <v-data-table
            :headers="[
              { text: this.$t('multilingual.project.overview.column.model'), value: 'model' },
              { text: this.$t('multilingual.project.overview.column.total'), value: 'total' },
              {
                text: this.$t('multilingual.project.overview.column.outdated'),
                value: 'text_outdated',
              },
              {
                text: this.$t('multilingual.project.overview.column.missing'),
                value: 'text_missing',
              },
              { text: this.$t('multilingual.project.overview.column.ok'), value: 'text_ok' },
              {
                text: this.$t('multilingual.project.overview.column.new'),
                value: 'reviewStatus.NEW',
              },
              {
                text: this.$t('multilingual.project.overview.column.auto'),
                value: 'reviewStatus.AUTO',
              },
              {
                text: this.$t('multilingual.project.overview.column.busy'),
                value: 'reviewStatus.BUSY',
              },
              {
                text: this.$t('multilingual.project.overview.column.request'),
                value: 'reviewStatus.COMPLETED',
              },
              {
                text: this.$t('multilingual.project.overview.column.denied'),
                value: 'reviewStatus.DENIED',
              },
              {
                text: this.$t('multilingual.project.overview.column.approved'),
                value: 'reviewStatus.APPROVED',
              },
              {
                text: this.$t('multilingual.project.overview.column.to_complete'),
                value: 'reviewStatus.TO_COMPLETE',
              },
              {
                text: this.$t('multilingual.project.overview.column.to_approve'),
                value: 'reviewStatus.TO_APPROVE',
              },
              {
                text: this.$t('multilingual.project.overview.column.redirect_label'),
                value: 'actions',
                sortable: false,
              },
            ]"
            :items="data"
            :items-per-page="10"
            class="elevation-1"
          >
            <template v-slot:[`item.actions`]="{ item }">
              <button type="button" class="btn btn-primary" @click="redirect(item)"
                id="project-update">
                {{ $t('multilingual.action.update') }}
              </button>
            </template>
          </v-data-table>
        </div>
      </v-app>
    </div>
  </div>
</template>

<script>
import IntoLocaleSwitcher from '../Shared/IntoLocaleSwitcher.vue';

export default {
  components: {
    IntoLocaleSwitcher,
  },

  props: {
    data: {
      type: Array,
      required: false,
    },

    toLang: {
      type: String,
      required: true,
    },

    project: null,
  },

  methods: {
    redirect(item) {
      window.location.href = item.redirectUrl;
    },
  },
};
</script>
