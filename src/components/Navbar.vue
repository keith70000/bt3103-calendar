<template>
  <nav>
    <v-snackbar v-model="snackbar" :timeout="4000" top color="success">
      <span>Awesome! You added a new actvity</span>
      <v-btn flat color="white" @click="snackbar=false">Close</v-btn>
    </v-snackbar>

    <v-toolbar flat app>
      <v-toolbar-side-icon v-if="isSignedIn" @click="drawer = !drawer" class="grey--text"></v-toolbar-side-icon>
      <v-toolbar-title class="text-uppercase grey--text">
        <span class="font-weight-light">{{code}}</span>
        <span>{{name}}</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>

      <v-flex class="right" justify="right">
        <v-btn v-if="!isSignedIn" text class="right">
          <router-link to="/signin">Sign in</router-link>
        </v-btn>
        <v-btn v-if="!isSignedIn" text class="right">
          <router-link to="/signup">Sign up</router-link>
        </v-btn>
        <v-btn v-if="isSignedIn" text class="right" v-on:click="signOut">Sign out</v-btn>
      </v-flex>
      <!-- dropdown menu -->
      <v-menu offset-y>
        <v-btn v-if="isSignedIn" flat slot="activator" color="grey">
          <v-icon left>expand_more</v-icon>
          <span>Menu</span>
        </v-btn>
        <v-list>
          <v-list-tile v-for="link in links" :key="link.text" router :to="link.route">
            <v-list-tile-title>{{ link.text }}</v-list-tile-title>
          </v-list-tile>
        </v-list>
      </v-menu>

      <!-- Include if we want to have authentication
      <v-btn flat color="grey">
        <span>Sign Out</span>
        <v-icon right>exit_to_app</v-icon>
      </v-btn>
      -->
    </v-toolbar>

    <v-navigation-drawer v-if="isSignedIn" app v-model="drawer" class="primary">
      <v-layout column align-center>
        <v-flex class="mt-5">
          <v-avatar size="100">
            <img class="text-lg-center" src="../assets/avatar.jpeg" />
          </v-avatar>
        </v-flex>
        <p class="white--text subheading mt-1">{{currentUser.displayName}}</p>
        <p class="white--text subheading mt-1">{{currentUser.email}}</p>
        <v-flex class="mt-4 mb-3">
          <!-- <Popup @projectAdded="snackbar = true" /> -->
        </v-flex>
      </v-layout>
      <v-list>
        <v-list-tile v-for="link in links" :key="link.text" router :to="link.route">
          <v-list-tile-action>
            <v-icon class="white--text">{{ link.icon }}</v-icon>
          </v-list-tile-action>
          <v-list-tile-content>
            <v-list-tile-title class="white--text">
              {{
              link.text
              }}
            </v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </v-navigation-drawer>
  </nav>
</template>

<script>
import Popup from "./Popup";
import firebase from "firebase/app";
import db from "@/fb";

export default {
  components: { Popup },
  data() {
    return {
      isSignedIn: false,
      currentUser: "",
      drawer: false,
      links: [
        { icon: "home", text: "Select Project", route: "/" },
        { icon: "dashboard", text: "Dashboard", route: "/dashboard" },
        { icon: "folder", text: "To-do List", route: "/todos" },
        { icon: "person", text: "Team", route: "/team" },
        { icon: "calendar_today", text: "Calendar", route: "/calendar" }
      ],
      snackbar: false,
      name: "",
      code: ""
    };
  },
  mounted() {
    var self = this;
    firebase.auth().onAuthStateChanged(function(user) {
      if (user) {
        // User is signed in.
        console.log(user);
        self.isSignedIn = true;
        self.currentUser = user;
      } else {
        // No user is signed in.
        console.log("no user hello");
      }
    });
    db.collection("masterProjectList")
      .doc(self.$store.state.selectedProject)
      .get()
      .then(doc => {
        self.name = doc.data().name;
        self.code = doc.data().code;
      });
  },
  computed: {
    projectName() {
      return this.$store.state.projectName;
    },
    projectCode() {
      return this.$store.state.projectCode;
    }
  },
  watch: {
    projectName(oldName, newName) {
      this.name = oldName;
    },
    projectCode(oldCode, newCode) {
      this.code = oldCode;
    }
  },
  methods: {
    signOut() {
      firebase
        .auth()
        .signOut()
        .then(() => {
          window.location.reload();
          this.$router.go({ path: this.$router.path });
        });
    }
  }
};
</script>

<style></style>
