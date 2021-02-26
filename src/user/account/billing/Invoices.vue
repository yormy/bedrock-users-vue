<template>
  <div class="container">
    <div class="card">
      <div class="card-body">
        <div>Invoices page</div>
        <table class="table">
          <thead>
            <tr>
              <th>Payment Date</th>
              <th>Amount</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="payment in payments" :key="payment.xid">
              <td>{{ payment.created_at }}</td>
              <td>{{ payment.symbol }} {{ payment.total_decimals }}</td>
              <td>
                <button-submit
                  :is-loading="isLoading"
                  v-if="payment.downloadable"
                  btnClass="btn btn-sm btn-primary"
                  @clicked="download(payment.xid, payment.download_as_filename)"
                >
                  <slot name="button-content">
                    <i class="fal fa-download"></i>
                  </slot>
                </button-submit>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  components: {},

  props: {
    payments: {
      type: Array,
      required: true,
    },
  },

  data() {
    return {
      isLoading: null,
    };
  },

  methods: {
    download(xid, downloadAsFilename) {
      this.isLoading = true;

      const url = this.route('user.account.billing.invoices.download', xid);

      this.$http
        .get(url, { responseType: 'blob' })
        .then(response => {
          this.forceFileDownload(response, `${downloadAsFilename}.pdf`);
        })
        .catch(() => {})
        .finally(() => {
          this.isLoading = false;
        });
    },

    forceFileDownload(response, title) {
      const url = window.URL.createObjectURL(new Blob([response.data]));
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', title);
      document.body.appendChild(link);
      link.click();
    },
  },
};
</script>
