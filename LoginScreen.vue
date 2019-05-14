<template>
  <v-container>
    <v-container fluid fill-height>
      <v-layout align-center justify-center>
        <v-flex xs12 sm8 md4>
          <v-card class="elevation-12">
            <!-- <v-jumbotron dark src="https://i2.wp.com/beerstation.sk/wp-content/uploads/2018/06/cropped-without-shadow-black.png?fit=235%2C250">
              </v-jumbotron>
              <label class="display-1">Management</label> -->
            <v-card-text>
              <v-form>
                <v-text-field prepend-icon="person" v-model="email" v-validate="'required|email'" :error-messages="errors.collect('email')" label="E-mail" data-vv-name="email" required />
                <v-text-field prepend-icon="lock" v-model="password" v-validate="'required'" :error-messages="errors.collect('password')" data-vv-name="password" label="Password" :type="isPasswordVisible ? 'text' : 'password'" :append-icon="isPasswordVisible ? 'visibility_off' : 'visibility'" @click:append="isPasswordVisible = !isPasswordVisible" required />
              </v-form>
            </v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn @click="submit">{{ $t("signIn") }}</v-btn>
            </v-card-actions>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>
  </v-container>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { GRAPHQL_TOKEN, APP_ROUTER } from "@/models/constants";

@Component
export default class LoginScreen extends Vue {

    private email: string = "";
    private password: string = "";
    private isPasswordVisible: string = "";

  public async submit() {
    const validation = await this.$validator.validateAll();

    if (validation) {
      this.$store.dispatch('authenticate', {
        email: this.email,
        password: this.password
      }).then(() => {
        this.$router.replace(APP_ROUTER.OVERVIEW.PATH)
      }).catch((error) => alert(error))
    }
  }
}
</script>
