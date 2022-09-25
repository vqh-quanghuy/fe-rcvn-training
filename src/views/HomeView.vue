<template>
  <v-app>
    <v-container>
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
        <template v-slot:extension>
          <v-tabs v-model="tab" align-with-title>
            <v-tabs-slider color="yellow"></v-tabs-slider>
            <v-tab href="#tab-1"> Users </v-tab>
            <v-tab href="#tab-2"> Customers </v-tab>
            <v-tab href="#tab-3"> Products </v-tab>
          </v-tabs>
        </template>
      </v-app-bar>
      <v-tabs-items v-model="tab" class="my-2">
        <v-tab-item value="tab-1">
          <UserList />
        </v-tab-item>
        <v-tab-item value="tab-2">
          <CustomerList />
        </v-tab-item>
        <v-tab-item value="tab-3">
          <ProductList />
        </v-tab-item>
      </v-tabs-items>
    </v-container>
  </v-app>
</template>

<script>
// @ is an alias to /src
import UserList from "@/components/UserList.vue";
import CustomerList from "@/components/CustomerList.vue";
import ProductList from "@/components/ProductList.vue";

export default {
  name: "HomeView",
  data() {
    return {
      tab: "tab-1",
    };
  },
  components: {
    UserList,
    CustomerList,
    ProductList,
  },
  methods: {
    async logout() {
      await this.$axios
        .post(
          `${this.$backendUrl}user/logout`,
          {},
          {
            headers: {
              "Content-Type": "application/json",
              Accept: "application/json",
              Authorization: "Bearer " + sessionStorage.getItem("access_token"),
            },
          }
        )
        .then((res) => {
          if (res.status === 200) {
            sessionStorage.removeItem("access_token");
            sessionStorage.removeItem("user_info");
            this.$router.push("login");
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
  computed: {
    userName() {
      return JSON.parse(sessionStorage.getItem("user_info")).name;
    },
  },
};
</script>
