<template>
  <div class="home">
    <h1>KStalk</h1>
    <button
      v-on:click="start"
      class="btn"
    >
      Googleでログイン
    </button>
  </div>
</template>

<script>
import { getAuth, GoogleAuthProvider, signInWithPopup } from "firebase/auth";
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'
import App from '../views/App.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/',
    name: 'App',
    component: App
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

const auth = getAuth();
const provider = new GoogleAuthProvider();

export default {
  name: "hello",
  router,
  methods: {
    start() {
      signInWithPopup(auth, provider)
      .then((result) => {
        console.log(result)
        this.$router.push({name: "App"});
      }).catch((error) => {
        console.log(error)
      });
    }
  }
}
</script>

<style>
@import url(../assets/cooldesign.css);
.home {
  text-align: center;
  padding-top: 40px;
}
h1 {
  font-size: 80px;
  font-weight: 100;
}
@media (prefers-color-scheme: dark) {
  h1 {
    color: white; 
  }
}
</style>