<template>
  <v-layout row align-center justify-center class="pt-3 mt-3">
    <v-flex xs12 md4>
      <v-card>
        <v-card-title class="headline">Login</v-card-title>
        <v-card-text>
          <p>Login to view and manage your guests.</p>
          <v-form>
            <v-container fluid class="pa-0">
              <v-layout row>
                <v-flex xs12>
                  <v-text-field
                    name="email"
                    label="Email"
                    id="email"
                    v-model="email"
                    @input="$v.email.$touch()"
                    @blur="$v.email.$touch()"
                    @focus="onInputClick"
                    @keyup.enter="submit"
                    :error-messages="emailErrors"
                  ></v-text-field>
                </v-flex>
              </v-layout>
              <v-layout row>
                <v-flex xs12>
                  <v-text-field
                    name="password"
                    label="Password"
                    type="password"
                    v-model="password"
                    @input="$v.password.$touch()"
                    @blur="$v.password.$touch()"
                    @focus="onInputClick"
                    @keyup.enter="submit"
                    :error-messages="passwordErrors"
                    id="password"
                  ></v-text-field>
                </v-flex>
              </v-layout>
              <p>Don't have an account? <router-link to="registration">Click here to register.</router-link></p>
            </v-container>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="submit">Login</v-btn>
        </v-card-actions>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
  import { validationMixin } from 'vuelidate'
  import { required } from 'vuelidate/lib/validators'
  export default {
    fetch () {
      return this.loadFB
    },
    data () {
      return {
        email: '',
        password: '',
        incorrect: false,
        incorrectemail: false,
        incorrectpassword: false,
        fbSignInParams: {
          scope: 'public_profile,email',
          return_scopes: true
        }
      }
    },
    methods: {
      onInputClick () {
        if (this.incorrect) {
          this.incorrect = false
          this.$v.$reset()
        }
      },
      async submit () {
        this.$v.$touch()
        if (!this.$v.$invalid) {
          try {
            await this.$auth.loginWith('local', {
              data: {
                email: this.email,
                password: this.password
              }
            })
          } catch (error) {
            if (error.response) {
              switch (error.response.status) {
                case this.$g('HTTP_ERROR_CODES').UNAUTHORIZED:
                  console.log('UNAUTHORIZED')
                  this.incorrectemail = true
                  this.incorrectpassword = true
                  this.incorrect = true
                  break
                default:
                  console.log(error)
              }
            }
          }
        }
      }
    },
    validations: {
      email: { required },
      password: { required }
    },
    computed: {
      emailErrors () {
        const errors = []
        if (!this.$v.email.$dirty) return errors
        !this.$v.email.required && errors.push('Email is required')
        if (this.incorrectemail) {
          errors.push('Incorrect email/password')
          this.incorrectemail = false
        }
        return errors
      },
      passwordErrors () {
        const errors = []
        if (!this.$v.password.$dirty) return errors
        !this.$v.password.required && errors.push('Password is required')
        if (this.incorrectpassword) {
          errors.push('')
          this.incorrectpassword = false
        }
        return errors
      }
    },
    mixins: [ validationMixin ]
  }
</script>
