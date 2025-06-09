<template>
  <AuthCard>
    <form @submit.prevent="onRegister" autocomplete="off">
      <h2 class="fw-bold mb-4 text-center">Create Your Account</h2>
      <div class="row g-3">
        <div class="col-md-6">
          <div class="mb-3">
            <label class="form-label" for="firstname">First Name</label>
            <input v-model="firstname" type="text" class="form-control form-control-lg" id="firstname" placeholder="First Name" required>
          </div>
        </div>
        <div class="col-md-6">
          <div class="mb-3">
            <label class="form-label" for="lastname">Last Name</label>
            <input v-model="lastname" type="text" class="form-control form-control-lg" id="lastname" placeholder="Last Name" required>
          </div>
        </div>
        <div class="col-md-6">
          <div class="mb-3">
            <label class="form-label" for="username">Username</label>
            <input v-model="username" type="text" class="form-control form-control-lg" id="username" placeholder="Username" required>
          </div>
          <div v-if="usernameError" class="text-danger small ms-1 mt-1">{{ usernameError }}</div>
        </div>
        <div class="col-md-6">
          <div class="mb-3">
            <label class="form-label" for="country">Country</label>
            <input v-model="country" type="text" class="form-control form-control-lg" id="country" placeholder="Country" required>
          </div>
        </div>
        <div class="col-12">
          <div class="mb-3">
            <label class="form-label" for="email">Email</label>
            <input v-model="email" type="email" class="form-control form-control-lg" id="email" placeholder="Email" required>
          </div>
        </div>
        <div class="col-md-6">
          <div class="mb-3">
            <label class="form-label" for="password">Password</label>
            <input v-model="password" type="password" class="form-control form-control-lg" id="password" placeholder="Password" required>
          </div>
          <div v-if="passwordError" class="text-danger small ms-1 mt-1">{{ passwordError }}</div>
        </div>
        <div class="col-md-6">
          <div class="mb-3">
            <label class="form-label" for="confirmPassword">Confirm Password</label>
            <input v-model="confirmPassword" type="password" class="form-control form-control-lg" id="confirmPassword" placeholder="Confirm Password" required>
          </div>
          <div v-if="confirmPasswordError" class="text-danger small ms-1 mt-1">{{ confirmPasswordError }}</div>
        </div>
      </div>
      <div v-if="error" class="alert alert-danger mt-3 mb-0 py-2">{{ error }}</div>
      <button type="submit" class="btn btn-success btn-lg w-100 mt-4" :disabled="loading">
        <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
        Create Account
      </button>
      <p class="mt-4 text-center" style="color:rgb(2, 2, 2);">
        Already have an account?
        <router-link to="/login" style="color: rgb(27, 44, 228);">Login here</router-link>
      </p>
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
const firstname = ref('');
const lastname = ref('');
const country = ref('');
const password = ref('');
const confirmPassword = ref('');
const email = ref('');
const error = ref('');
const loading = ref(false);

const usernameError = ref('');
const passwordError = ref('');
const confirmPasswordError = ref('');

function validateUsername() {
  if (!/^[A-Za-z]{3,8}$/.test(username.value)) {
    usernameError.value = "Username must be 3-8 letters only.";
    return false;
  }
  usernameError.value = "";
  return true;
}

function validatePassword() {
  if (
    password.value.length < 5 ||
    password.value.length > 10 ||
    !/\d/.test(password.value) ||
    !/[!@#$%^&*(),.?":{}|<>]/.test(password.value)
  ) {
    passwordError.value = "Password must be 5-10 chars, include a number and a special character.";
    return false;
  }
  passwordError.value = "";
  return true;
}

function validateConfirmPassword() {
  if (password.value !== confirmPassword.value) {
    confirmPasswordError.value = "Passwords do not match.";
    return false;
  }
  confirmPasswordError.value = "";
  return true;
}

function validateAll() {
  let valid = true;
  if (!validateUsername()) valid = false;
  if (!validatePassword()) valid = false;
  if (!validateConfirmPassword()) valid = false;
  if (!firstname.value || !lastname.value || !country.value || !email.value) valid = false;
  if (!/^[^@]+@[^@]+\.[^@]+$/.test(email.value)) {
    error.value = "Invalid email address.";
    valid = false;
  } else {
    error.value = "";
  }
  return valid;
}

async function onRegister() {
  error.value = "";
  if (!validateAll()) return;
  loading.value = true;
  try {
    // Log server URL
    console.log("Server URL:", store.server_domain);

    const response = await axios.post(`${store.server_domain}/Register`, {
      username: username.value,
      firstname: firstname.value,
      lastname: lastname.value,
      country: country.value,
      password: password.value,
      email: email.value,
      profilePic: null
    }, {
      // Add headers and timeout
      headers: {
        'Content-Type': 'application/json'
      },
      timeout: 5000,
      validateStatus: function (status) {
        return status >= 200 && status < 500;
      }
    });

    console.log("Registration response:", response);
    
    if (response.status === 201 || response.status === 200) {
      // Use the global toast function
      app.appContext.config.globalProperties.toast("Registration Successful", "You can now login.", "success");
      router.push("/login");
    } else {
      throw new Error(response.data?.message || 'Registration failed');
    }
  } catch (e) {
    console.error("Registration error details:", {
      message: e.message,
      response: e.response,
      config: e.config
    });

    if (e.code === 'ECONNABORTED') {
      error.value = "Connection timeout. Please try again.";
    } else if (e.message.includes('Network Error')) {
      error.value = "Cannot connect to server. Please check your connection and try again.";
    } else if (e.response?.data?.message) {
      error.value = e.response.data.message;
    } else if (e.response?.status === 409) {
      error.value = "Username already exists. Please choose another.";
    } else {
      error.value = `Registration failed: ${e.message}`;
    }
  } finally {
    loading.value = false;
  }
}
</script>

<style>
/* Global styles */
body {
  background-color: #e6f4ea !important;
}
</style>

<style scoped>
.auth-container {
  height: calc(100vh - 56px);
}

.image-side {
  width: 600px;
  height: 600px;
  margin: auto 0;
  margin-left: 2rem;
}

.recipe-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 8px;
}

.form-side {
  flex: 1;
  padding: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
}

.card {
  background: transparent;
  width: 100%;
  max-width: 600px;
}

.card-body form {
  text-align: center;
}

.form-label, .form-control {
  text-align: left;
}
</style>
