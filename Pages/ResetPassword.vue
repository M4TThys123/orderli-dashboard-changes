<template>
  <div>
    <!-- Header -->
    <div class="header py-6 py-lg-1 pt-lg-9" style="background: linear-gradient(87deg, #d65729 0px, #d64629 100%) !important;">
      <div class="container">
        <div class="header-body text-center mb-7">
          <div class="row justify-content-center">
            <div class="col-xl-5 col-lg-6 col-md-8 px-5">
              <!--              <h1 class="text-white">Create an account</h1>-->
              <!--              <p class="text-lead text-white">Use these awesome forms to login or create new account in your project for-->
              <!--                free.</p>-->
            </div>
          </div>
        </div>
      </div>
      <div class="separator separator-bottom separator-skew zindex-100">
        <svg x="0" y="0" viewBox="0 0 2560 100" preserveAspectRatio="none" version="1.1"
             xmlns="http://www.w3.org/2000/svg">
          <polygon fill="#f8f9fe" points="3800 0 3800 100 0 100"></polygon>
        </svg>
      </div>
    </div>
    <!-- Page content -->
    <div class="container mt--8 pb-0">
      <!-- Table -->
      <div class="row justify-content-center">
        <div class="col-lg-6 col-md-8">
          <div class="card bg-secondary border-0 mb-0">
            <div class="card-body px-lg-5 py-lg-5">
              <article class="text-center text-muted mb-4">
                <h1 v-show="!linkExpired">Password reset</h1>
                <small v-show="!linkExpired">Please choose a new password </small>

                <h1 v-if="linkExpired">Password reset link expired</h1>
                <small v-if="linkExpired">You can login or reset your password again</small>
              </article>

              <nav class="text-center" v-if="linkExpired">
                <router-link class="btn btn-primary mt-4" to="/login">Login</router-link>
                <router-link class="btn btn-primary mt-4" to="/forgot-password">Reset password</router-link>
              </nav>

              <validation-observer v-slot="{handleSubmit}" ref="formValidator" v-show="!linkExpired">
                <form role="form" @submit.prevent="handleSubmit(onSubmit)">
                  <base-input alternative
                              class="mb-3"
                              prepend-icon="ni ni-lock-circle-open"
                              placeholder="Password"
                              type="password"
                              name="Password"
                              autocomplete="new-password"
                              :rules="{required: true, min: 6}"
                              v-model="model.password">
                  </base-input>
                  <div class="text-center">
                    <button type="submit" class="btn btn-primary mt-4">Reset password</button>
                  </div>
                </form>
              </validation-observer>
            </div>
          </div>
          <div class="row mt-3">
            <div class="col-6">
              <router-link to="/forgot-password" class="text-light"><small>Forgot password?</small></router-link>
            </div>
            <div class="col-6 text-right">
              <router-link to="/login" class="text-light"><small>Login</small></router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import swal from 'sweetalert2'
import 'sweetalert2/dist/sweetalert2.css'

export default {
  name: 'ResetPassword',
  data() {
    return {
      model: {
        password: '',
      },
      formSubmitted: false,
      linkExpired: false,
    }
  },
  computed: {
    token(){return this.$route.params.id}
  },
  mounted() {
    this.$store.dispatch('validateReset', this.token)
      .then(function(){
        console.log('valid invite token');
      })
      .catch(err => {
        swal.fire({
          icon: 'error',
          title: 'Oops....',
          text: err.response?.data.message || err,
          customClass: {
            confirmButton: 'btn btn-primary btn-fill'
          }

        })
        this.linkExpired = true
      })
  },
  methods: {
    onSubmit() {
      // this will be called only after form is valid. You can do an api call here to register users
      let resetData = {
        'token': this.token,
        'data': {
          'password': this.model.password,
          'confirm_password': this.model.password
        }
      }

      let self = this;

      this.$store.dispatch('resetPassword', resetData)
        .then(function(){
          self.formSubmitted = true;
          self.model.password = "";
          self.$refs.formValidator.reset();
          self.$router.push("/")
        })
        .catch(function(err) {
          swal.fire({
            icon: 'error',
            title: 'Oops....',
            text: err.response?.data.message,
            customClass: {
              confirmButton: 'btn btn-primary btn-fill'
            }
          })
        })
    }
  }

};
</script>
<style></style>
