<template>
  <AuthCard>
    <form @submit.prevent="onLogin" autocomplete="off">
      <h2 class="fw-bold mb-4 text-center">Sign in to your account</h2>
      <div class="mb-4">
        <label class="form-label" for="username">Username</label>
        <input v-model="username" type="text" id="username" class="form-control form-control-lg" placeholder="Username" required />
      </div>
      <div class="mb-4">
        <label class="form-label" for="password">Password</label>
        <input v-model="password" type="password" id="password" class="form-control form-control-lg" placeholder="Password" required />
      </div>
      <div v-if="error" class="alert alert-danger mt-2 mb-0 py-2">{{ error }}</div>
      <div class="pt-1 mb-4">
        <button class="btn btn-success btn-lg btn-block w-100" type="submit" :disabled="loading">
          <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
          Login
        </button>
      </div>
      <!-- <a class="small text-muted" href="#!">Forgot password?</a> -->
      <p class="mb-5 pb-lg-2" style="color: rgb(2, 2, 2);">
        Don't have an account?
        <router-link to="/register" style="color:rgb(27, 44, 228);">Register here</router-link>
      </p>
      <!-- <a href="#!" class="small text-muted">Terms of use.</a>
      <a href="#!" class="small text-muted">Privacy policy</a> -->
    </form>
  </AuthCard>
</template>

<script setup>
import { ref, inject, getCurrentInstance } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';
import AuthCard from '../components/AuthCard.vue';

const store = inject('store');
const router = useRouter();
const app = getCurrentInstance();

const username = ref('');
const password = ref('');
const error = ref('');
const loading = ref(false);

async function onLogin() {
  error.value = "";
  if (!username.value || !password.value) {
    error.value = "Please enter username and password.";
    return;
  }
  loading.value = true;
  try {
    await axios.post(`${store.server_domain}/Login`, {
      username: username.value, // must be lowercase to match backend
      password: password.value
    });
    store.login(username.value);
    app.appContext.config.globalProperties.toast("Login Successful", `Welcome, ${username.value}!`, "success");
    router.push("/");
  } catch (e) {
    error.value = e.response?.data?.message || "Login failed.";
  } finally {
    loading.value = false;
  }
}
</script>

<style scoped>
.form-label, .form-control {
  text-align: left;
}
</style>
