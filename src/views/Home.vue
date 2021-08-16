<template>
  <v-sheet id="home-sheet">
    <v-card max-width="800px" class="ma-auto py-4">
      <v-card-title>Login to start scheduling</v-card-title>
      <v-card-text>
        <v-form ref="form" v-model="valid" lazy-validation>
          <v-text-field
            v-model="username"
            label="Username"
            :rules="[usernameRules]"
          ></v-text-field>
          <v-text-field
            v-model="password"
            label="Password"
            type="password"
            :rules="[passwordRules]"
          ></v-text-field>
        </v-form>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="accent" @click="signup">Sign-up</v-btn>
        <v-btn color="primary" @click="login">Login</v-btn>
      </v-card-actions>
    </v-card>
    <v-snackbar v-model="snackbar" timeout="4000">{{ message }}</v-snackbar>
  </v-sheet>
</template>

<script>
import { endpoints } from "../endpoints";

export default {
  name: "Home",
  data() {
    return {
      username: "",
      password: "",
      valid: false,
      snackbar: false,
      message: "",
    };
  },
  methods: {
    login() {
      if (!this.$refs.form.validate()) {
        return;
      }

      const params = { username: this.username, password: this.password };
      try {
        this.axios
          .post(endpoints.login, params)
          .then((res) => {
            localStorage.setItem("schedulerJWT", res.data.token);
            this.$router.push({ path: "/Scheduler" });
          })
          .catch((err) => {
            this.message = `Error: ${err.response.status}: ${err.response.data}`;
            this.$nextTick(() => {
              this.snackbar = true;
            });
          });
      } catch {
        this.message = "Connection Error";
        this.$nextTick(() => {
          this.snackbar = true;
        });
      }
    },
    signup() {
      if (!this.$refs.form.validate()) {
        return;
      }
      const params = { username: this.username, password: this.password };
      this.axios
        .post(endpoints.signup, params)
        .then((res) => {
          localStorage.setItem("schedulerJWT", res.data.token);
          this.$router.push({ path: "/Scheduler" });
        })
        .catch((err) => {
          this.message = `Error: ${err.response.status}: ${err.response.data}`;
          this.$nextTick(() => {
            this.snackbar = true;
          });
        });
    },
    validate() {
      return (
        this.usernameRules(this.username) === true &&
        this.passwordRules(this.password) === true
      );
    },
    usernameRules(v) {
      return !!v || "Username is required";
    },
    passwordRules(v) {
      return !!v || "Password is required";
    },
  },
};
</script>
