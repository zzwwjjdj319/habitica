<template lang="pug">
.form
  .form-group.row.text-center
    .col-12.col-md-6
      .btn.btn-secondary.social-button(@click='socialAuth("facebook")')
        .svg-icon.social-icon(v-html="icons.facebookIcon")
        span {{registering ? $t('signUpWithSocial', {social: 'Facebook'}) : $t('loginWithSocial', {social: 'Facebook'})}}
    .col-12.col-md-6
      .btn.btn-secondary.social-button(@click='socialAuth("google")')
        .svg-icon.social-icon(v-html="icons.googleIcon")
        span {{registering ? $t('signUpWithSocial', {social: 'Google'}) : $t('loginWithSocial', {social: 'Google'})}}
  .form-group(v-if='registering')
    label(for='usernameInput', v-once) {{$t('username')}}
    input#usernameInput.form-control(type='text', :placeholder='$t("usernamePlaceholder")', v-model='username')
  .form-group(v-if='!registering')
    label(for='usernameInput', v-once) {{$t('emailOrUsername')}}
    input#usernameInput.form-control(type='text', :placeholder='$t("emailOrUsername")', v-model='username')
  .form-group(v-if='registering')
    label(for='emailInput', v-once) {{$t('email')}}
    input#emailInput.form-control(type='email', :placeholder='$t("emailPlaceholder")', v-model='email')
  .form-group
    label(for='passwordInput', v-once) {{$t('password')}}
    a.float-right.forgot-password(v-once, v-if='!registering', @click='forgotPassword = true') {{$t('forgotPassword')}}
    input#passwordInput.form-control(type='password', :placeholder='$t(registering ? "passwordPlaceholder" : "password")', v-model='password')
  .form-group(v-if='registering')
    label(for='confirmPasswordInput', v-once) {{$t('confirmPassword')}}
    input#confirmPasswordInput.form-control(type='password', :placeholder='$t("confirmPasswordPlaceholder")', v-model='passwordConfirm')
    small.form-text(v-once, v-html="$t('termsAndAgreement')")
  .text-center
    .btn.btn-info(@click='register()', v-if='registering', v-once) {{$t('joinHabitica')}}
    .btn.btn-info(@click='login()', v-if='!registering', v-once) {{$t('login')}}
</template>

<style lang="scss" scoped>
  @import '~client/assets/scss/colors.scss';

  .form {
    margin: 0 auto;
    width: 100%;
    padding-top: 2em;
    padding-bottom: 4em;
    position: relative;
    z-index: 1;

    .form-group {
      text-align: left;
      font-weight: bold;
    }

    .social-button {
      width: 100%;
      text-align: center;

      .text {
        display: inline-block;
        min-height: 0px;
      }
    }

    .social-icon {
      margin-right: 1em;
      width: 18px;
      height: 18px;
      display: inline-block;
      vertical-align: top;
      margin-top: .2em;
    }

    small.form-text {
      text-align: center;
    }
  }
</style>

<script>
import hello from 'hellojs';
import { setUpAxios } from 'client/libs/auth';

import facebookSquareIcon from 'assets/svg/facebook-square.svg';
import googleIcon from 'assets/svg/google.svg';

export default {
  name: 'AuthForm',
  data () {
    let data = {
      registering: true,
      username: '',
      email: '',
      password: '',
      passwordConfirm: '',
    };

    data.icons = Object.freeze({
      facebookIcon: facebookSquareIcon,
      googleIcon,
    });

    return data;
  },
  mounted () {
    hello.init({
      facebook: process.env.FACEBOOK_KEY, // eslint-disable-line
      // windows: WINDOWS_CLIENT_ID,
      google: process.env.GOOGLE_CLIENT_ID, // eslint-disable-line
    });
  },
  methods: {
    async socialAuth (network) {
      let auth = await hello(network).login({
        scope: 'email',
        redirect_uri: '', // eslint-disable-line camelcase
      });

      await this.$store.dispatch('auth:socialAuth', {
        auth,
      });

      await this.finishAuth();
    },
    async register () {
      if (!this.email) {
        alert(this.$t('missingEmail'));
        return;
      }

      if (this.password !== this.passwordConfirm) {
        alert(this.$t('passwordConfirmationMatch'));
        return;
      }

      try {
        await this.$store.dispatch('auth:register', {
          username: this.username,
          email: this.email,
          password: this.password,
          passwordConfirm: this.passwordConfirm,
        });

        await this.finishAuth();
      } catch (e) {
        if (e.response.data.data && e.response.data.data.errors) {
          const message = e.response.data.data.errors.map(error => `${error.message}\n`);
          alert(message);
        }
      }
    },
    async finishAuth () {
      setUpAxios();

      await this.$store.dispatch('user:fetch', {forceLoad: true});

      this.$emit('authenticate');
    },
  },
};
</script>
