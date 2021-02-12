<template>
  <div>
    <h1>{{$t('multilingual.project.settings.title') | capitalizeFirst}}</h1>
    <ValidationObserver ref="form">
      <ValidationProvider v-slot="{ errors }" rules="required|min:2" name="project_name">

        <v-text-field
          name="project_name"
          id="project_name"
          :label="$t('multilingual.project.name') | capitalizeFirst"
          v-model="projectName"
          outlined
          :error-messages="apiErrors.project_name ? apiErrors.project_name : errors"
          :color="!apiErrors.project_name && projectName ? 'success' : ''"
          @keydown="apiErrors.project_name = ''"
        >
        </v-text-field>
      </ValidationProvider>

      <button-submit :is-loading="isSaving" @clicked="updateProject">
        {{ $t('multilingual.action.save') | capitalizeFirst }}
      </button-submit>
    </ValidationObserver>

    <div v-if="table.messages.success" class="alert alert-success">
      {{ table.messages.success }}
    </div>
    <!--    <div v-if="table.messages.error" class="alert alert-danger">-->
    <!--      {{ table.messages.error }}-->
    <!--    </div>-->

    <div v-if="table.messages.apiErrors" class="alert alert-danger">
      <ul>
        <li v-for="apiErrorField in table.messages.apiErrors" :key="apiErrorField">
          <div v-for="apiError in apiErrorField" :key="apiError">{{ apiError }}</div>
        </li>
      </ul>
    </div>

    <div class="col-4">
      <v-card>
        <v-row>
          <v-col class="ml-1">
            {{ $t('multilingual.project.settings.whitelist_ip.title') | capitalizeFirst}}
          </v-col>

          <v-col class="text-right mr-1">
            <button-add-modal
              @addItem="addItem"
              :isLoading="table.state.isSubmittingAdd"
              btn-class=""
              header-class="green"
              :add-text="$t('multilingual.action.add')"
              :cancel-text="$t('multilingual.action.cancel')"
            >
              <template v-slot:header>{{ $t('multilingual.action.add') | capitalizeFirst }}</template>
              <template v-slot:form>
                <h3>
                  {{
                    $t('multilingual.project.settings.whitelist_ip.title') | capitalizeFirst
                  }}
                </h3>
                <p>
                  {{
                    $t('multilingual.project.settings.whitelist_ip.description')
                      | capitalizeFirst
                  }}
                </p>
                <ValidationObserver ref="form">
                  <ValidationProvider v-slot="{ errors }" rules="required|min:2" name="ip_address">
                    <v-text-field
                      name="ip_address"
                      id="ip_address"
                      v-model="table.formAdd.ip"
                      :label="$t('multilingual.misc.ip_address')"
                      outlined
                      :error-messages="
                        table.messages.apiErrors && table.messages.apiErrors.ip
                          ? table.messages.apiErrors.ip
                          : table.formAdd.ip !== null
                          ? errors
                          : ''
                      "
                      :color="
                        !table.messages.apiErrors ||
                        (!table.messages.apiErrors.ip && table.formAdd.ip)
                          ? 'success'
                          : ''
                      "
                      @keydown="table.messages.apiErrors ? (table.messages.apiErrors.ip = '') : ''"
                    >
                    </v-text-field>
                  </ValidationProvider>
                </ValidationObserver>
              </template>
            </button-add-modal>
          </v-col>
        </v-row>

        <v-data-table
          :headers="table.headers"
          :items="table.values"
          :items-per-page="10"
          item-key="id"
          single-select
          hide-default-header
          class="elevation-1"
        >
          <template v-slot:[`item.actions`]="{ item }">
            <div class="text-right">
              <button-delete
                :item="item"
                @deleteItem="deleteAgreed(item)"
                btn-class="btn btn-link text-danger"
              >
                <template v-slot:delete-preview>
                  <div class="mt-5">
                    {{ item.ip }}
                  </div>
                </template>
              </button-delete>
            </div>
          </template>
        </v-data-table>
      </v-card>
    </div>
  </div>
</template>

<script>
import { Datatable, ButtonDelete, ButtonAddModal } from 'bedrock-vue-components';

export default {
  extends: Datatable,

  components: {
    ButtonDelete,
    ButtonAddModal,
  },

  props: {
    project: {
      type: Object,
      required: true,
    },

    toLang: {
      type: String,
      required: true,
    },
  },

  data() {
    return {
      table: {
        headers: null,
        values: this.project.ip_whitelist,
        state: {
          isSubmittingRemove: false,
          isSubmittingAdd: false,
        },
        messages: {
          success: '',
          error: '',
          apiErrors: null,
        },
        formAdd: {
          ip: '',
        },
      },

      projectName: this.project.name,

      isSaving: false,

      apiErrors: [],
    };
  },

  created() {
    this.setHeaders();
  },

  methods: {
    setHeaders() {
      this.table.headers = [
        {
          // text: this.$t('multilingual-admin.label.group'),
          text: 'ip',
          value: 'ip',
        },
        {
          // text: this.$t('multilingual-admin.label.group'),
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

    isValidProjectSettings() {
      this.$refs.form.validate();
      return this.$refs.form.flags.valid;
    },

    updateProject() {
      if (!this.isValidProjectSettings()) {
        return;
      }

      this.isSaving = true;
      const successUrl = this.route('user.multilingual.projects.show', [
        this.project.xid,
        this.toLang,
      ]);
      const updateUrl = this.route('api.v1.user.multilingual.projects.update', this.project.xid);

      const data = {
        project_name: this.projectName,
      };

      const self = this;
      this.$http
        .put(updateUrl, data)
        .then(() => {
          window.location.href = successUrl;
        })
        .catch((error) => {
          self.apiErrors = error.response.data.errors;
          self.isSaving = false;
        })
        .finally(() => {});
    },

    newItem() {
      return {
        id: this.table.formAdd.ip,
        ip: this.table.formAdd.ip,
        dummy: '@',
      };
    },

    addItem() {
      this.clearResponses();
      this.table.state.isSubmittingAdd = true;
      this.table.values = this.addItemToTable(this.table.values, this.newItem());

      const url = this.route('api.v1.user.multilingual.projects.add_ip', this.project.xid);
      const payload = this.table.formAdd;

      this.$http
        .post(url, payload)
        .then(() => {
          this.clearFormAdd();
        })
        .catch((error) => {
          this.table.values = this.restoreTable();
          this.table.messages.error = error.response.data.message;
          this.table.messages.apiErrors = error.response.data.errors;
        })
        .finally(() => {
          this.table.state.isSubmittingAdd = false;
        });
    },

    deleteAgreed(item) {
      this.table.values = this.deleteItemFromTable(this.table.values, item, 'id');
      this.deleteModal = false;
      this.deleteItem(item);
    },

    deleteItem(item) {
      this.clearResponses();
      this.table.state.isSubmittingRemove = true;

      const url = this.route('api.v1.user.multilingual.projects.delete_whitelisted_ip', [
        this.project.xid,
        item.id,
      ]);

      this.$http
        .delete(url)
        .then(() => {})
        .catch(() => {
          this.table.values = this.restoreTable();
        })
        .finally(() => {
          this.table.state.isSubmittingRemove = false;
        });
    },

    clearResponses() {
      this.table.messages.success = '';
      this.table.messages.error = '';
      this.table.messages.apiErrors = null;
    },

    clearFormAdd() {
      this.table.formAdd.ip = null;
    },
  },
};
</script>

<style scoped>
::v-deep .v-data-footer {
  padding-left: 0px;
  padding-right: 0px;
}
</style>
