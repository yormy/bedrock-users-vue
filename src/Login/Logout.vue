<template>
  <span>
    <button :class="buttonClass" :disabled="isDisabled()" @click="clicked">
      <span v-if="iconPrepend" :class="iconPrepend"></span>

      <span v-if="isLoading">
        <span class="fa fa-spinner fa-spin"></span>
        <slot name="loading-title">Loading...</slot>
      </span>
      <span v-else>
        <slot></slot>
      </span>

      <span v-if="iconAppend" :class="iconAppend"></span>
    </button>
  </span>
</template>

<script>
import { doLogout } from 'bedrock-vue-components';

export default {
  props: {
    buttonClass: {
      type: String,
    },

    url: {
      type: String,
      required: true,
    },

    redirect: {
      type: String,
    },

    iconPrepend: {
      type: String,
    },

    iconAppend: {
      type: String,
    },

    disabled: {
      type: Boolean,
    },

    isLoading: {
      type: Boolean,
    },
  },

  methods: {
    isDisabled() {
      return this.disabled || this.isLoading;
    },

    clicked() {
      if (!this.isDisabled()) {
        doLogout(this.$http);
        this.$emit('clicked');
      }
    },
  },
};
</script>

<style scoped></style>
