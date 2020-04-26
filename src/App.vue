<template>
  <v-app class="grey lighten-3">
    <navbar/>
    <v-content>
      <keep-alive>
      <router-view></router-view>
      </keep-alive>
    </v-content>
  </v-app>
</template>

<script>
import navbar from './components/Navbar'

export default {
  name: 'App',
  components: {
    navbar
  },

  data: () => ({
    //
  }),
};
</script>
