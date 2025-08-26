<template>
  <b-container>
    <h3 class="title-section">
      <strong>{{ title }}:</strong>
      <slot></slot>
    </h3>
    <div v-if="loading" class="text-center py-4">
      <div class="spinner-border text-success" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>
    <div v-else-if="error" class="alert alert-danger">
      {{ error }}
    </div>
    <div v-else-if="recipes.length === 0" class="empty-state text-center py-5">
      <i class="fas" :class="getEmptyIcon()" fa-3x mb-3 text-muted></i>
      <h3>{{ getEmptyTitle() }}</h3>
      <p class="text-muted mb-4">{{ emptyMessage }}</p>
      <router-link v-if="source === 'favorites'" to="/" class="btn btn-primary">
        Explore Recipes
      </router-link>
      <button v-else-if="source === 'my-recipes'" class="btn btn-primary" @click="$emit('create-recipe')">
        Create Your First Recipe
      </button>
    </div>
    <div v-else class="recipes-container">
      <RecipePreview
        v-for="r in recipes"
        :key="r.id"
        class="recipe-item"
        :recipe="r"
      />
    </div>
  </b-container>
</template>

<script>
import RecipePreview from "./RecipePreview.vue";
export default {
  name: "RecipePreviewList",
  components: {
    RecipePreview
  },
  props: {
    title: {
      type: String,
      required: true
    },
    source: {
      type: String,
      default: "random", // 'random', 'viewed', 'favorites', 'my-recipes'
    },
    emptyMessage: {
      type: String,
      default: "No recipes to display."
    }
  },
  emits: ['create-recipe'],
  data() {
    return {
      recipes: [],
      loading: true,
      error: null
    };
  },
  mounted() {
    this.updateRecipes();
  },
  methods: {
    getEmptyIcon() {
      switch (this.source) {
        case 'favorites': return 'fa-heart';
        case 'my-recipes': return 'fa-book';
        case 'viewed': return 'fa-eye';
        default: return 'fa-utensils';
      }
    },
    getEmptyTitle() {
      switch (this.source) {
        case 'favorites': return 'No Favorite Recipes';
        case 'my-recipes': return 'No Created Recipes';
        case 'viewed': return 'No Viewed Recipes';
        default: return 'No Recipes';
      }
    },
    async updateRecipes() {
      this.loading = true;
      this.error = null;
      
      try {
        let endpoint;
        switch (this.source) {
          case "viewed":
            endpoint = `${this.$root.store.server_domain}/users/viewed-recipes`;
            break;
          case "favorites":
            endpoint = `${this.$root.store.server_domain}/users/favorites`;
            break;
          case "my-recipes":
            endpoint = `${this.$root.store.server_domain}/users/my-recipes`;
            break;
          case "random":
          default:
            endpoint = `${this.$root.store.server_domain}/recipes/random`;
        }
        
        const response = await this.axios.get(endpoint, { withCredentials: true });
        console.log(`${this.source} recipes response:`, response.data);
        
        this.recipes = [];
        
        if (response.data && Array.isArray(response.data)) {
          this.recipes.push(...response.data);
        }
      } catch (error) {
        console.error(`Error fetching ${this.source} recipes:`, error);
        if (error.response?.status === 401 && this.source !== "random") {
          // For non-random sources that require authentication
          this.error = "Please log in to view this content.";
        } else {
          this.error = `Failed to load recipes. ${error.message}`;
        }
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.title-section {
  text-align: center;
  margin: 2rem 0 3rem;
  font-weight: bold;
  font-size: 1.8rem;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 15px;
}

.recipes-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
  justify-items: center;
  margin: 0 auto;
  max-width: 1200px;
}

.recipe-item {
  width: 100%;
  max-width: 350px;
}

.empty-state {
  background-color: white;
  border-radius: 10px;
  padding: 2rem;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

/* Responsive breakpoints */
@media (min-width: 1200px) {
  .recipes-container {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (max-width: 991px) {
  .recipes-container {
    grid-template-columns: repeat(2, 1fr);
    gap: 15px;
  }
}

@media (max-width: 576px) {
  .recipes-container {
    grid-template-columns: 1fr;
    gap: 15px;
  }
  
  .recipe-item {
    max-width: 100%;
  }
}
</style>
