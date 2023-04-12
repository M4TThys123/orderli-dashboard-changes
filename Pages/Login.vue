<template>
  <div>
    <!-- Header -->
    <div class="header py-7 py-lg-8 pt-lg-9" style="background: linear-gradient(87deg, #d65729 0px, #d64629 100%) !important;">
      <div class="container">
        <div class="header-body text-center mb-7 mb-lg-4">
          <div class="row justify-content-center">
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
      <div class="row justify-content-center">
        <div class="col-lg-5 col-md-7">
          <div class="card bg-secondary border-0 mb-0">
            <div class="card-body px-lg-5 py-lg-5">
              <div class="text-center text-muted mb-4">
                <h1>Login</h1>
                <small>Login to your Orderli account</small>
              </div>
              <validation-observer v-slot="{handleSubmit}" ref="formValidator">
                <form role="form" @submit.prevent="handleSubmit(onSubmit)">
                  <base-input alternative
                              class="mb-3"
                              name="Email"
                              :rules="{required: true, email: true}"
                              autocomplete="username"
                              prepend-icon="ni ni-email-83"
                              placeholder="Email"
                              v-model.trim="model.email">
                  </base-input>

                  <base-input alternative
                              class="mb-3"
                              name="Password"
                              :rules="{required: true}"
                              autocomplete="current-password"
                              prepend-icon="ni ni-lock-circle-open"
                              :type="showPassword ? 'text' : 'password'"
                              placeholder="Password"
                              v-model="model.password">
                    <i slot="append" :class="showPassword ? 'fas fa-eye-slash' : 'fas fa-eye'" @click="togglePassword"></i>
                  </base-input>

                  <base-checkbox v-model="model.rememberMe">Stay logged in for 7 days</base-checkbox>
                  <div class="text-center">
                    <base-button type="primary" native-type="submit" class="my-4" :disabled="authStatus === 'loading'">Sign in</base-button>
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
              <router-link to="/register" class="text-light"><small>Create a new account</small></router-link>
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

  //TODO add show password toggle button to password field

  export default {
    name: "Login",
    data() {
      return {
        model: {
          email: '',
          password: '',
          rememberMe: true
        },
        showPassword: false,
        failedAttempts: 0,
      };
    },
    computed: {
      authStatus() {return this.$store.state.auth.status},
    },
    methods: {
      togglePassword() {
        this.showPassword = !this.showPassword;
      },
      onSubmit() {
        // this will be called only after form is valid. You can do api call here to login
        let loginData = {
          'email': this.model.email,
          'password': this.model.password,
          'remember': this.model.rememberMe
        }

        let self = this;

        this.$store.dispatch('login', loginData)
        .then(function() {self.$router.push(self.$route.query.nextUrl || "/")})
        .catch(err => {
          self.failedAttempts++
          if (self.failedAttempts >= 2) {
            swal.fire({
              icon: 'warning',
              title: 'Forgot your password?',
              text: 'You have entered a wrong password twice. Would you like to reset your password?',
              showCancelButton: true,
              confirmButtonText: 'Reset Password',
              cancelButtonText: 'Cancel',
              customClass: {
                confirmButton: 'btn btn-primary btn-fill',
                cancelButton: 'btn btn-danger btn-fill'
              }
            }).then((result) => {
              if (result.isConfirmed) {
                // redirect to password reset page
                self.$router.push('/forgot-password');
              }
            });
          }
          else{
            swal.fire({
              icon: 'error',
              title: 'Oops....',
              text: err.response?.data.message || err,
              customClass: {
                confirmButton: 'btn btn-primary btn-fill'
              }
            })
          }
          },
        )
      }
    }
  };
</script>
