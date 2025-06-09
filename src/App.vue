<template>
  <div>
    <Navbar />
    <main class="container mt-4">
      <router-view />
    </main>
  </div>
</template>

<script>
import { getCurrentInstance } from 'vue';
import Navbar from './components/Navbar.vue';

export default {
  components: { Navbar },
  name: "App",
  setup() {
    const internalInstance = getCurrentInstance();
    const store = internalInstance.appContext.config.globalProperties.store;
    const toast = internalInstance.appContext.config.globalProperties.toast;
    const router = internalInstance.appContext.config.globalProperties.$router;

    const logout = () => {
      store.logout();
      toast("Logout", "User logged out successfully", "success");
      router.push("/").catch(() => {});
    };

    return { store, logout };
  }
}
</script>

<style lang="scss">
@import "@/scss/form-style.scss";

body {
  margin: 0;
  min-height: 100vh;
  background-color: #e6f4ea !important;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  min-height: 100vh;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}

/* Remove background color from individual pages since it's now global */
.auth-container, .about-container {
  background-color: transparent;
}
</style>
