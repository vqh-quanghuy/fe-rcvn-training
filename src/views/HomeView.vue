<template>
  <v-app>
    <div class="home">
      <nav>
        <router-link to="/home">Home</router-link> |
        <router-link to="/about">About</router-link>
      </nav>  
      <UserList />
    </div>
  </v-app>
</template>

<script>
// @ is an alias to /src
import UserList from '@/components/UserList.vue'

export default {
  name: 'HomeView',
  components: {
    UserList
}
}
</script>
