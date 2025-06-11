<template>
  <v-app dark>
    <!-- Left Navigation Drawer -->
    <v-navigation-drawer
      v-if="$route.name !== 'login'"
      v-model="drawer"
      app
      clipped
      color="grey darken-4"
      class="elevation-4"
    >
      <v-list dense nav>
        <v-list-item class="pa-3">
          <v-img src="/logo.jpeg" height="50" contain />
        </v-list-item>
        <v-divider></v-divider>
        <v-list-item
          v-for="(item, i) in items"
          :key="i"
          :to="item.to"
          router
          exact
          link
          active-class="deep-purple--text text--accent-2"
        >
          <v-list-item-icon>
            <v-icon color="white">{{ item.icon }}</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title class="white--text">{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <!-- App Bar -->
    <v-app-bar app flat color="white" height="64" class="elevation-2">
      <v-container fluid>
        <v-row align="center" no-gutters>
          <!-- Logo -->
          <v-col cols="auto">
            <v-img src="/logo.jpeg" contain height="48" width="100" class="pointer" @click="goIndex" />
          </v-col>
          <!-- Spacer -->
          <v-spacer></v-spacer>
          <!-- Login/Logout Buttons -->
          <v-col cols="auto">
            <v-btn
              v-if="!$auth.loggedIn"
              text
              class="text--primary"
              @click="pushRoute('login')"
            >
              Login
            </v-btn>
            <v-btn
              v-if="$auth.loggedIn"
              color="deep-purple accent-4"
              dark
              depressed
              @click="$auth.logout()"
            >
              Logout
            </v-btn>
          </v-col>
        </v-row>
      </v-container>
    </v-app-bar>

    <!-- Main Content -->
    <v-main>
      <v-container fluid class="pa-4">
        <Nuxt />
      </v-container>
    </v-main>

    <!-- Optional Right Drawer -->
    <v-navigation-drawer
      v-model="rightDrawer"
      :right="right"
      temporary
      fixed
      color="grey lighten-4"
    >
      <v-list>
        <v-list-item @click="right = !right">
          <v-list-item-icon>
            <v-icon>mdi-repeat</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>Switch drawer (click me)</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
  </v-app>
</template>
<script>
export default {
  name: "DefaultLayout",
  data() {
    return {
      drawer: true,
      rightDrawer: false,
      right: true,
      clipped: true,
      miniVariant: false,
      items: [
        { icon: "mdi-view-dashboard", title: "Dashboard", to: "/dashboard" },
        { icon: "mdi-file-document-outline", title: "Logs", to: "/logs" },
        { icon: "mdi-account-multiple", title: "Recipients", to: "/recepients" },
      ],
    }
  },
  methods: {
    pushRoute(link) {
      this.$router.push({ path: `/${link}` });
    },
    goIndex() {
      this.$router.push('/');
    },
  },
}
</script>
<style scoped>
.pointer {
  cursor: pointer;
}
</style>
