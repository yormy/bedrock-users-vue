<template>
  <div>
    <div id="labels_overview_table">
      <v-card>
        <v-row>
          <v-col>
            <div class="text-right">
              <button type="submit" class="btn btn-success btn-sm"
                      id="add-item" v-if="canAdd()" @click="addItem">
                +
              </button>
            </div>
          </v-col>
        </v-row>

        <v-row>
          <v-col class="pt-0 pb-0">
            <list-review-status-component
              :review-status-selected.sync="reviewStatusSelected"
            ></list-review-status-component>
          </v-col>

          <v-col class="pt-0 pb-0">
            <list-text-status-component
              :text-status-selected.sync="textStatusSelected"
            ></list-text-status-component>
          </v-col>

          <v-spacer></v-spacer>

          <v-col class="pt-0 pb-0" cols="4">
            <list-search-component :search-input.sync="searchInput"></list-search-component>
          </v-col>
        </v-row>

        <v-data-table
          :headers="headers"
          :items="state.items"
          :search="searchData"
          :custom-filter="filter"
          :items-per-page="10"
          @click:row="rowClick"
          @current-items="getFiltered"
          item-key="meta.id"
          single-select
          class="elevation-1"
          show-expand
          v-model="selectedItems"
        >
          <template v-slot:expanded-item="{ headers, item }">
            <td :colspan="headers.length">
              {{ $t('multilingual.misc.id') }}: {{ item.meta.id }} <br />
              <span class="status-icon" :class="item.meta.text_status"></span>
              {{ $t(`multilingual.status.textstatus.status.${item.meta.text_status}.text`) }}<br />

              <span class="status-icon" :class="item.meta.review_status"></span>
              {{ $t(`multilingual.status.reviewstatus.status.${item.meta.review_status}.text`)}}
              <br />
              {{ item }}
            </td>
          </template>

          <!--            <template v-slot:item.meta.text_status="{ item }">-->
          <!--                <td style="display: none"></td>-->
          <!--            </template>-->
          <!--            <template v-slot:item.meta.review_status="{ item }">-->
          <!--                <td style="display: none"></td>-->
          <!--            </template>-->
          <!--            <template v-slot:item.dummy="{ item }">-->
          <!--                <td style="background-color: red;
          padding-left: 0px !important; padding-right: 0px !important; "></td>-->
          <!--            </template>-->

          <template v-slot:[`item.icons`]="{ item }">
            <div class="d-flex">
              <span class="status-icon" :class="item.meta.text_status"></span>
              <span class="status-icon" :class="item.meta.review_status"></span>
            </div>
          </template>

        </v-data-table>
      </v-card>
    </div>
  </div>
</template>

<script>
import BaseList from '../Base/BaseList.vue';
import ListReviewStatusComponent from '../Base/ListReviewStatusComponent.vue';
import ListTextStatusComponent from '../Base/ListTextStatusComponent.vue';
import ListSearchComponent from '../Base/ListSearchComponent.vue';

export default {
  extends: BaseList,

  components: {
    ListReviewStatusComponent,
    ListTextStatusComponent,
    ListSearchComponent,
  },

  methods: {
    setHeadersNonBase() {
      this.setHeadersBaseLanguage();
    },

    setHeadersBaseLanguage() {
      this.headers = [
        {
          text: this.$t('multilingual.label.property.group'),
          value: 'staticFields.group',
        },
        {
          text: this.$t('multilingual.label.property.key'),
          value: 'staticFields.key',
          sortable: true,
          filterable: true,
        },
        {
          text: this.$t('multilingual.label.property.text'),
          value: 'fields.text.currentTranslated',
        },
        {
          text: '',
          value: 'icons',
          filterable: false,
          sortable: false,
        },
        // {text: $t('multilingual.status.shared.actions'), value: 'actions', sortable: false },
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

<style lang="scss">

$check: '\f00c';
$bullhorn: '\f0a1';
$hourglass-half: '\f252';
$ban: '\f05e';

$exclamation :'\f12a';
$question-circle: '\f059';
$edit: '\f044';
$check-circle : '\f058';
$redo: '\f01e';

.status-icon {
  font-weight: 600;
  font-family: "Font Awesome 5 Free";
}

/* ----- TEXT ICONS ---- */
.TRANSLATION_STATUS_TEXT_OK {
  position:relative;
  color:#32A852;

  &::before {
    content: $check;
  }
}

.TRANSLATION_STATUS_TEXT_MISSING {
  position:relative;
  color:#A81D1B;

  &::before {
    content: $bullhorn
  }
}

.TRANSLATION_STATUS_TEXT_OUTDATED {
  position:relative;
  color:#B46B26;

  &::before {
    content: $hourglass-half;
  }
}

.TRANSLATION_STATUS_TEXT_BASE_LANG {
  position:relative;
  color:#32A852;

  &::before {
    content: $check;
  }
}

/* ----- REVIEW ICONS ---- */
.TRANSLATION_STATUS_REVIEW_AUTO {
  position:relative;
  color:#1BA8A6;

  &::before {
    content: $bullhorn;
  }
}

.TRANSLATION_STATUS_REVIEW_BUSY {
  position:relative;
  color:#A6A820;

  &::before {
    content: $hourglass-half;
  }
}

.TRANSLATION_STATUS_REVIEW_NEW {
  position:relative;
  color:#A81D1B;

  &::before {
    content: $bullhorn;
  }
}

.TRANSLATION_STATUS_REVIEW_APPROVED {
  position:relative;
  color:#32A852;

  &::before {
    content: $check;
  }
}

.TRANSLATION_STATUS_REVIEW_DENIED {
  position:relative;
  color:#A81D1B;

  &::before {
    content: $ban;
  }
}

.TRANSLATION_STATUS_REVIEW_REQUEST {
  position:relative;
  color:#A6A820;

  &::before {
    content: $check;
  }
}

/* ----- COMBINED ICONS ---- */
.TRANSLATION_STATUS_COMBINED_NEW {
  position:relative;
  color:#A81D1B;

  &::before {
    content: $exclamation;
  }
}

.TRANSLATION_STATUS_COMBINED_MISSING {
  position:relative;
  color:#A81D1B;

  &::before {
    content: $question-circle;
  }
}

.TRANSLATION_STATUS_COMBINED_OUTDATED {
  position:relative;
  color:#A6A820;

  &::before {
    content: $edit;
  }
}

.TRANSLATION_STATUS_COMBINED_DONE {
  position:relative;
  color:#32A852;

  &::before {
    content: $check-circle;
  }
}

.TRANSLATION_STATUS_COMBINED_BUSY {
  position:relative;
  color:#1BA8A6;

  &::before {
    content: $hourglass-half;
  }
}

.TRANSLATION_STATUS_COMBINED_DENIED {
  position:relative;
  color:#A81D1B;

  &::before {
    content: $ban;
  }
}

.TRANSLATION_STATUS_COMBINED_UNDER_REVIEW {
  position:relative;
  color:#1BA8A6;

  &::before {
    content: $redo;
  }
}

</style>
