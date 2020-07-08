<template>
  <section>
    <div class="container">
      <div class="modal-card" style="width: auto">
        <header class="modal-card-head">
          <p class="modal-card-title">Login</p>
        </header>
        <section class="modal-card-body">
        <b-field label="Username">
          <b-input v-model="username" maxlength="30"></b-input>
        </b-field>

        <b-field label="Password">
          <b-input v-model="password" type="password" maxlength="30"></b-input>
        </b-field>
        <b-button @click="login">Login</b-button>
        </section>

        
      </div>
    </div>
  </section>
</template>

<script>
export default {
  data() {
    return {
      username: "",
      password: ""
    };
  },
  methods: {
    login() {
      if (
        this.username == process.env.USER &&
        this.password == process.env.PASS
      ) {
        this.$store.commit("auth");
        this.$buefy.notification.open("Logged in");
        let redirect_url = "/";
        this.$router.push(redirect_url);
      } else {
        this.$buefy.notification.open({message: "Username and password incorrect", type:"is-warning"});
        this.username = "";
        this.password = "";
      }
    }
  }
};
</script>