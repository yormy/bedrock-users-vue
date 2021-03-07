<template>
  <div>
    <v-dialog v-model="popupWarning" width="500" persistent>
      <v-card>
        <v-card-title class="headline">{{ $t('bedrock-users.login.autologout.warning.title')}}</v-card-title>

        <v-card-text>
          <p v-html="$t('bedrock-users.login.autologout.warning.description', {timeleft: timeLeft})"></p>
          <button @click="prolongSession" class="btn btn-success">{{ $t('bedrock-users.login.autologout.warning.button')}}</button>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import AUTH from '@config/auth';
import { isLoggedIn, doLogout } from 'bedrock-vue-components';

export default {
  props: {
    loggedInAs: {
      required: true,
    },
  },

  data() {
    return {
      events: ['click', 'mousemove', 'mousedown', 'scroll', 'keypress', 'load'],
      warningTimer: null,
      logoutTimer: null,
      countdownTimer: null,
      popupWarning: false,
      subscription: null,
      isLoggedIn: true,
      timeLeft: '',
      // isLoggedIn: this.$store.state.auth.isUserLoggedIn(), //??
    };
  },

  created() {
    this.createListeners();
    // if (isLoggedIn(this.loggedInAs)) {
    //   this.createListeners();
    // }
  },

  beforeDestroy() {
    this.subscription();
  },

  mounted() {
    if (isLoggedIn(this.loggedInAs)) {
      this.createListeners();
    }
  },

  destroyed() {
    this.destroyListeners();
  },

  methods: {
    prolongSession() {
      this.resetTimer();
    },

    createListeners() {
      // listen for mousemoves etc
      // this.events.forEach((event) => {
      //   window.addEventListener(event, this.resetTimer);
      // });
      //
      this.setTimers();
    },

    destroyListeners() {
      this.events.forEach(event => {
        window.removeEventListener(event, this.resetTimer);
      });

      this.clearTimer();
    },

    setTimers() {
      // if (window.$cookies.get('rememberMe')) {
      //   this.logoutTimer = setTimeout(
      //     this.logoutUser,
      //     this.inMinutes(AUTH.AUTO_LOGOUT_REMEMBER_ME_HOURS),
      //   );
      // } else {
      this.logoutTimer = setTimeout(this.logoutUser, this.inMinutes(AUTH.AUTO_LOGOUT_MINUTES));
      //      }

      this.warningTimer = setTimeout(this.warningMessage, this.getWarningTimeout());
    },

    startCountdownTimer(durationMs) {
      let timer = durationMs / 1000;
      let minutes;
      let seconds;
      const that = this;
      setInterval(() => {
        minutes = parseInt(timer / 60, 10);
        seconds = parseInt(timer % 60, 10);

        minutes = minutes < 10 ? `0${minutes}` : minutes;
        seconds = seconds < 10 ? `0${seconds}` : seconds;

        timer -= 1;
        if (timer < 0) {
          minutes = '00';
          seconds = '00';
        }

        that.timeLeft = `${minutes}:${seconds}`;
      }, 1000);
    },

    // getWarningTimeout() {
    //   if (window.$cookies.get('rememberMe')) {
    //     return (
    //       this.inHours(AUTH.AUTO_LOGOUT_REMEMBER_ME_HOURS) -
    //       this.inMinutes(AUTH.AUTO_LOGOUT_WARN_IN_ADVANCE_MINUTES)
    //     );
    //   }
    //   return (
    //     this.inMinutes(AUTH.AUTO_LOGOUT_MINUTES) -
    //     this.inMinutes(AUTH.AUTO_LOGOUT_WARN_IN_ADVANCE_MINUTES)
    //   );
    // },
    getWarningTimeout() {
      return (
        this.inMinutes(AUTH.AUTO_LOGOUT_MINUTES) -
        this.inMinutes(AUTH.AUTO_LOGOUT_WARN_IN_ADVANCE_MINUTES)
      );
    },

    inMinutes(minutes) {
      return minutes * 60 * 1000;
    },

    inHours(hours) {
      return this.inMinutes(hours * 60);
    },

    warningMessage() {
      this.popupWarning = true;
      this.startCountdownTimer(this.getWarningTimeout());
    },

    resetTimer() {
      this.clearTimer();
      this.popupWarning = false;
      this.setTimers();
    },

    clearTimer() {
      clearTimeout(this.warningTimer);
      clearTimeout(this.logoutTimer);
    },

    reLogin() {
      // console.log('relogin');
    },

    logoutUser() {
      this.destroyListeners();
      this.popupWarning = false;

      const currentUrl = window.location.href;
      doLogout(this.loggedInAs, this.$http, currentUrl);
    },
  },
};
</script>
