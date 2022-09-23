<template>
  <v-app>
    <v-app-bar color="primary" dark absolute elevate-on-scroll height="50px">
      <v-toolbar-title>Coding Beauty</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-title>{{ userName }}</v-toolbar-title>
      <v-menu left bottom>
        <template v-slot:activator="{ on, attrs }">
          <v-btn icon v-bind="attrs" v-on="on" @click="logout()">
            <v-icon>mdi-logout</v-icon>
          </v-btn>
        </template>
      </v-menu>
    </v-app-bar>
    <v-container class="mt-12">
      <UserList />
    </v-container>
  </v-app>
</template>

<script>
// @ is an alias to /src
import UserList from '@/components/UserList.vue'

export default {
  name: 'HomeView',
  components: {
    UserList
  },
  methods: {
    async logout() {
      await this.$axios
      .post(`${this.$backendUrl}user/logout`, {}, {
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
          'Authorization': 'Bearer ' + sessionStorage.getItem("access_token")
        }
      })
      .then(res => {
        if (res.status === 200) {
          sessionStorage.removeItem("access_token");
          sessionStorage.removeItem("user_info");
          this.$router.push('login');
        }
      })
      .catch(err => {
        console.log(err);
      })
    }
  },
  computed: {
    userName() {
      return JSON.parse(sessionStorage.getItem('user_info')).name
    }
  }
}
</script>
