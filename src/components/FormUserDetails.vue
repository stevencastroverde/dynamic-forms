<template>
  <div>
    <h1 class="title">Create Account</h1>

    <h2 class="subtitle">
      Create an account or log in to order your liquid gold subscription
    </h2>

    <form  v-if="loggedIn" class="form" @input="submit">
      <div class="form-group">
        <label class="form-label" for="email">Email</label>
        <input type="text" @blur="checkIfUserExists" v-model="$v.form.email.$model" placeholder="your@email.com" class="form-control" id="email">
        <div v-if="$v.form.email.$error && !$v.form.email.required" class="error">email is required</div>
        <div v-if="$v.form.email.$error && !$v.form.email.email" class="error">email is invalid</div>
      </div>


      <div v-if="emailCheckedInDB" class="form-group">
        <label class="form-label" for="password">Password</label>
        <input v-model="$v.form.password.$model" type="password" placeholder="Super Secret Password" class="form-control" id="password">
        <div v-if="$v.form.password.$error && !$v.form.password.required" class="error">password is required</div>
      </div>
      <div v-if="existingUser" class="form-group">
        <button class="btn">Login</button>
      </div>


      <div v-if="emailCheckedInDB && ! existingUser" class="form-group">
        <label class="form-label" for="name">Name</label>
        <input v-model="$v.form.name.$model" type="text" placeholder="What should we call you?" class="form-control" id="name">
        <div v-if="$v.form.name.$error" class="error">name is required</div>
      </div>
    </form>
    <div class="text-center">
      Successfully Logged in! <a @click="resetForm" href="#">Not {{form.name}}?</a>
    </div>
  </div>
</template>

<script>
  import {authenticateUser, checkIfUserExistsInDB} from '../api';
  import {required, email} from 'vuelidate/lib/validators'
  export default {
    data () {
      return {
        form: {
          email: null,
          password: null,
          name: null,
        },
        emailCheckedInDB: false,
        existingUser: false,
        wrongPassword: false
      }
    },
    validations: {
      form: {
        email: {
          required,
          email
        },
        password: {
          required,
          correct () {
            return ! this.wrongPassword
          }
        },
        name: {
          required
        }
      }
    },
    methods: {
      submit() {
          if (! this.$v.$invalid) {
            this.$emit('update', {
             data: {
                email: this.form.email,
                password: this.form.password,
                name: this.form.name
             },
             valid: !this.$v.$invalid
            })
          }
      },
      computed: {
        loggedIn () {
          return this.existingUser && this.form.name;
        }
      },
      checkIfUserExists() {
        if (!this.$v.form.email.$invalid) {
          this.$emit('updateAsyncState', 'pending');
          return checkIfUserExistsInDB(this.form.email)
          .then(() => {
            this.existingUser = true;
            this.emailCheckedInDB = true;
            this.$emit('updateAsyncState', 'success');
          })
          .catch( () => {
            this.existingUser = false;
            this.checkIfUserExistsInDB = true;
            this.$emit('updateAsyncState', 'failure');
          })
        }
      },
      login() {
        this.wrongPassword = false;
        if (!this.$v.form.password.$invalid) {
          this.$emit('updateAsyncState', 'pending');
          return authenticateUser(this.form.email, this.form.password)
          .then( user => {
            this.wrongPassword = false;
            this.form.name = user.name;
            this.$emit('updateAsyncState', 'success');
            this.submit();
          })
          .catch(() => {
            this.wrongPassword = true;
            this.$emit('updateAsyncState', 'failure');
          })
        }
      },
      resetForm () {
        this.form.email = null;
        this.form.password = null;
        this.form.name = null;
        this.emailCheckedInDB = false;
        this.wrongPassword = false;
        this.existingUser = false;
        this.$v.$reset();
      }
    }
  }
</script>

<style scoped>

</style>