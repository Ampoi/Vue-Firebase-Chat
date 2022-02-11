<template>
  <router-view/>
</template>

<script>
import { initializeApp } from "firebase/app";
import { getAnalytics,logEvent } from "firebase/analytics";
// 以下firebase
const firebaseConfig = {
  apiKey: "***",
  authDomain: "***",
  projectId: "***",
  storageBucket: "***",
  messagingSenderId: "***",
  appId: "***",
  measurementId: "***"
};

const app = initializeApp(firebaseConfig);

const analytics = getAnalytics(app);
logEvent(analytics, 'notification_received');

export default {
  name: "app"
}
</script>