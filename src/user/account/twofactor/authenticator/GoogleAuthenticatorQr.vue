<template>
  <div>
    <span v-if="!qrCode" class="fa fa-3x fa-spinner fa-spin"></span>
    <img id="qr_code" :src="qrCode" alt="" style="height: 200px" />
  </div>
</template>

<script>
export default {
  data() {
    return {
      errorMessage: '',
      successMessage: '',
      qrCode: '',
    };
  },

  created() {
    this.$http
      .get(this.qrUrl)
      .then(success => {
        const response = success.data;
        if (response.success === false) {
          this.errorMessage = response.message;
        } else {
          const { data } = response;
          this.qrCode = data.qr;
          this.$emit('loaded');
        }
      })
      .catch(() => {
        this.errorMessage = 'error';
      });
  },

  props: {
    qrUrl: {
      type: String,
      required: true,
    },
  },
};
</script>
