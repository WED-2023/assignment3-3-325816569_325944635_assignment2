<template>
  <div class="container">
    <h1 class="title text-center mb-5">Welcome to RecipeApp</h1>

    <div class="row">
      <!-- Left Column -->
      <div class="col-md-6 left-column">
        <RecipePreviewList title="Explore these recipes" class="random-recipes" ref="randomRecipes" />
        <div class="text-center mt-4">
          <button @click="refreshRandomRecipes" class="btn btn-primary">
            <i class="fas fa-sync-alt"></i> Get New Recipes
          </button>
        </div>
      </div>

      <!-- Right Column -->
      <div class="col-md-6 right-column">
        <template v-if="store.username">
          <RecipePreviewList
            title="Last watched recipes"
            class="last-viewed-recipes"
          />
        </template>
        <template v-else>
          <div class="login-section text-center">
            <h3>Join Our Community!</h3>
            <p class="mb-4">Log in to track your favorite recipes and see your viewing history.</p>
            <router-link :to="{ name: 'login' }" class="btn btn-success btn-lg mb-3">
              Login
            </router-link>
            <p>Don't have an account?</p>
            <router-link :to="{ name: 'register' }" class="btn btn-outline-primary">
              Register Now
            </router-link>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import { getCurrentInstance } from 'vue';
import RecipePreviewList from "../components/RecipePreviewList.vue";

export default {
  components: {
    RecipePreviewList
  },
  setup() {
    const internalInstance = getCurrentInstance();
    const store = internalInstance.appContext.config.globalProperties.store;

    return { store };
  },
  methods: {
    refreshRandomRecipes() {
      this.$refs.randomRecipes.updateRecipes();
    }
  }
};
</script>

<style lang="scss" scoped>
.container {
  padding: 2rem 0;
}

.row {
  margin: 0 -15px;
}

.left-column, .right-column {
  padding: 0 25px;
}

.login-section {
  background: #f8f9fa;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.random-recipes, .last-viewed-recipes {
  margin-bottom: 2rem;
}
</style>
