<template>
  <v-app>
    <!-- <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <h1>
          かんりまる
        </h1>
      </div>

      <v-spacer></v-spacer>
            <router-link to="/">Home</router-link> |
            <router-link to="/about">About</router-link>
    </v-app-bar> -->

    
          <router-view/>
    
  </v-app>
</template>

<script>

export default {
  name: 'App',
  data: () => ({
    //
  }),
  };
</script>
