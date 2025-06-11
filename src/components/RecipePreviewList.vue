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
    <div v-else-if="recipes.length === 0" class="alert alert-info text-center">
      {{ emptyMessage }}
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
      default: "random", // 'random', 'viewed', 'favorites', etc.
    },
    emptyMessage: {
      type: String,
      default: "No recipes to display."
    }
  },
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
  display: flex;
  align-items: flex-start;
  gap: 20px;
  margin: 0 auto;
}

.recipe-item {
  flex: 0 0 calc(33.333% - 14px);
}
</style>
