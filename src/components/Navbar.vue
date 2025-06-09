<template>
  <nav class="navbar navbar-expand-lg">
    <div class="container">
      <router-link class="navbar-brand d-flex align-items-center" to="/">
        <i class="fas fa-utensils me-2"></i>
        <span>RecipeApp</span>
      </router-link>
      
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
        <span class="navbar-toggler-icon"></span>
      </button>

      <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav me-auto">
          <li class="nav-item">
            <router-link class="nav-link" to="/">
              <i class="fas fa-home me-1"></i>Recipes
            </router-link>
          </li>
          <li class="nav-item">
            <router-link class="nav-link" to="/search">
              <i class="fas fa-search me-1"></i>Search
            </router-link>
          </li>
          <li class="nav-item">
            <router-link class="nav-link" to="/about">
              <i class="fas fa-info-circle me-1"></i>About
            </router-link>
          </li>
          <li v-if="store.username" class="nav-item">
            <router-link class="nav-link" to="/create-recipe">
              <i class="fas fa-plus-circle me-1"></i>Create Recipe
            </router-link>
          </li>
        </ul>

        <ul class="navbar-nav">
          <template v-if="!store.username">
            <li class="nav-item">
              <span class="nav-link">
                <i class="fas fa-user me-1"></i>Hello Guest
              </span>
            </li>
            <li class="nav-item">
              <router-link class="nav-link btn btn-outline-light ms-2" to="/login">Login</router-link>
            </li>
            <li class="nav-item">
              <router-link class="nav-link btn btn-outline-light ms-2" to="/register">Register</router-link>
            </li>
          </template>

          <template v-else>
            <li class="nav-item">
              <span class="nav-link">
                <i class="fas fa-user me-1"></i>Hello {{ store.username }}
              </span>
            </li>
            <li class="nav-item dropdown">
              <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                <i class="fas fa-user-circle me-1"></i>Personal Area
              </a>
              <ul class="dropdown-menu dropdown-menu-end">
                <li>
                  <router-link class="dropdown-item" to="/favorites">
                    <i class="fas fa-heart me-2"></i>My Favorites
                  </router-link>
                </li>
                <li>
                  <router-link class="dropdown-item" to="/my-recipes">
                    <i class="fas fa-book me-2"></i>My Recipes
                  </router-link>
                </li>
                <li>
                  <router-link class="dropdown-item" to="/family-recipes">
                    <i class="fas fa-users me-2"></i>Family Recipes
                  </router-link>
                </li>
              </ul>
            </li>
            <li class="nav-item">
              <a class="nav-link btn btn-outline-light ms-2" href="#" @click.prevent="logout">
                <i class="fas fa-sign-out-alt me-1"></i>Logout
              </a>
            </li>
          </template>
        </ul>
      </div>
    </div>
  </nav>
</template>

<script setup>
import { inject } from 'vue';
const store = inject('store') || window.store;
function logout() {
  store.logout();
}
</script>

<script>
export default {
  name: "AppNavbar"
};
</script>

<style scoped>
.navbar {
  background-color: #4caf50;
  padding: 0.5rem 1rem;
}

.navbar-brand {
  color: white;
  font-weight: bold;
  font-size: 1.4rem;
}

.nav-link {
  color: rgba(255, 255, 255, 0.9);
  padding: 0.5rem 1rem;
  transition: all 0.3s ease;
}

.nav-link:hover {
  color: white;
}

.btn-outline-light {
  border-color: rgba(255, 255, 255, 0.5);
}

.dropdown-menu {
  background-color: white;
  border: none;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
}

.dropdown-item {
  color: #4caf50;
}

.dropdown-item:hover {
  background-color: #e6f4ea;
}

.navbar-toggler {
  border-color: rgba(255, 255, 255, 0.5);
}

.navbar-toggler-icon {
  background-image: url("data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 30 30'><path stroke='rgba(255, 255, 255, 0.9)' stroke-linecap='round' stroke-miterlimit='10' stroke-width='2' d='M4 7h22M4 15h22M4 23h22'/></svg>");
}
</style>
