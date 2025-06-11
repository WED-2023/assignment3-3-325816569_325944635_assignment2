<template>
  <div class="container mt-4">
    <!-- Loading State -->
    <div v-if="loading" class="text-center py-5">
      <div class="spinner-border text-success" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      <p class="mt-3">Loading recipe details...</p>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="alert alert-danger">
      <i class="fas fa-exclamation-circle me-2"></i>
      {{ error }}
      <div class="mt-3">
        <button @click="$router.push('/')" class="btn btn-outline-danger">
          <i class="fas fa-home me-2"></i>Return to Home
        </button>
      </div>
    </div>

    <!-- Recipe Details -->
    <div v-else-if="recipe" class="recipe-container">
      <div class="recipe-header mb-4">
        <h1 class="recipe-title">{{ recipe.title }}</h1>
        
        <div class="recipe-badges mb-3">
          <span v-if="recipe.vegan" class="badge bg-success me-2">Vegan</span>
          <span v-if="recipe.vegetarian" class="badge bg-success me-2">Vegetarian</span>
          <span v-if="recipe.glutenFree" class="badge bg-warning me-2">Gluten-Free</span>
          <span v-if="recipe.viewed" class="badge bg-info me-2">Viewed</span>
        </div>
        
        <div class="recipe-image-container">
          <img :src="recipe.image" class="recipe-image" :alt="recipe.title" />
        </div>
      </div>
      
      <div class="row recipe-info-bar p-3 mb-4">
        <div class="col-md-3 text-center">
          <i class="fas fa-clock me-2"></i>
          <span class="fw-bold">Ready in:</span> {{ recipe.readyInMinutes }} minutes
        </div>
        <div class="col-md-3 text-center">
          <i class="fas fa-utensils me-2"></i>
          <span class="fw-bold">Servings:</span> {{ recipe.servings || 4 }}
        </div>
        <div class="col-md-3 text-center">
          <i class="fas fa-heart me-2"></i>
          <span class="fw-bold">Likes:</span> {{ recipe.popularity || 0 }}
        </div>
        <div class="col-md-3 text-center">
          <button 
            @click="addToFavorites" 
            class="btn btn-sm"
            :class="{ 'btn-success': !localFavorite, 'btn-secondary': localFavorite }"
            :disabled="localFavorite || !store.username"
          >
            <i class="fas" :class="localFavorite ? 'fa-heart' : 'fa-heart-o'"></i>
            {{ localFavorite ? 'In Favorites' : 'Add to Favorites' }}
          </button>
        </div>
      </div>
      
      <div class="row">
        <div class="col-md-6">
          <div class="recipe-section">
            <h2 class="section-title">
              <i class="fas fa-list me-2"></i>Ingredients
            </h2>
            <ul class="ingredients-list">
              <li v-for="(ingredient, index) in recipe.ingredients" :key="index">
                {{ ingredient.name }} ({{ ingredient.amount }})
              </li>
            </ul>
          </div>
        </div>
        
        <div class="col-md-6">
          <div class="recipe-section">
            <h2 class="section-title">
              <i class="fas fa-utensils me-2"></i>Instructions
            </h2>
            <!-- Use v-html for HTML content -->
            <div v-if="instructionsHaveHtml" v-html="recipe.steps"></div>
            <ol v-else class="instructions-list">
              <li v-for="(step, index) in instructionSteps" :key="index">
                {{ step }}
              </li>
            </ol>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      recipe: null,
      loading: true,
      error: null,
      store: this.$root.store,
      localFavorite: false
    };
  },
  computed: {
    instructionsHaveHtml() {
      return this.recipe && 
             typeof this.recipe.steps === 'string' && 
             (this.recipe.steps.includes('<ol>') || this.recipe.steps.includes('<ul>'));
    },
    instructionSteps() {
      if (!this.recipe || !this.recipe.steps) {
        return ["No instructions available"];
      }
      
      // Check if it's HTML content
      if (this.instructionsHaveHtml) {
        return []; // We'll use v-html instead
      }
      
      if (typeof this.recipe.steps === 'string') {
        // Try to parse JSON if it's a string
        try {
          return JSON.parse(this.recipe.steps);
        } catch (e) {
          return [this.recipe.steps];
        }
      } else if (Array.isArray(this.recipe.steps)) {
        return this.recipe.steps;
      } else if (this.recipe.analyzedInstructions && this.recipe.analyzedInstructions.length > 0) {
        return this.recipe.analyzedInstructions[0].steps.map(s => s.step);
      }
      
      return ["No instructions available"];
    }
  },
  watch: {
    // Update when recipe changes
    'recipe.favorite': {
      handler(newVal) {
        this.localFavorite = newVal || false;
      },
      immediate: true
    }
  },
  created() {
    this.fetchRecipeDetails();
  },
  methods: {
    async fetchRecipeDetails() {
      this.loading = true;
      this.error = null;
      
      try {
        const recipeId = this.$route.params.recipeId;
        
        // Get the is_DB parameter directly from the query
        const is_DB = this.$route.query.is_DB === 'true' || this.$route.query.is_DB === true;
        
        console.log(`Fetching recipe ${recipeId}, is_DB: ${is_DB}`);
        
        let response;
        if (is_DB) {
          // For database recipes
          response = await axios.get(
            `${this.store.server_domain}/recipes/DB/${recipeId}`,
            { withCredentials: true }
          );
        } else {
          // For Spoonacular recipes
          response = await axios.get(
            `${this.store.server_domain}/recipes/${recipeId}`,
            { withCredentials: true }
          );
        }
        
        console.log("Recipe details response:", response.data);
        this.recipe = response.data;
        
        // Explicitly set the is_DB property
        this.recipe.is_DB = is_DB;
        
        // Check if this recipe is in favorites
        if (this.store.username) {
          await this.checkFavoriteStatus();
        }
      } catch (error) {
        console.error("Error fetching recipe details:", error);
        this.error = error.response?.data?.message || "Failed to load recipe details";
      } finally {
        this.loading = false;
      }
    },
    
    async checkFavoriteStatus() {
      try {
        const recipeId = this.$route.params.recipeId;
        const is_DB = this.recipe.is_DB;
        
        const response = await axios.get(
          `${this.store.server_domain}/users/favorites/${recipeId}`,
          { 
            params: { is_DB },
            withCredentials: true 
          }
        );
        
        if (response.data.isFavorite) {
          this.localFavorite = true;
          this.recipe.favorite = true;
        }
      } catch (error) {
        console.error("Error checking favorite status:", error);
      }
    },
    
    async addToFavorites() {
      if (!this.store.username || this.localFavorite) {
        return;
      }
      
      try {
        const recipeId = this.$route.params.recipeId;
        const is_DB = this.recipe.is_DB;
        
        await axios.post(
          `${this.store.server_domain}/users/favorites`, 
          {
            recipeId: recipeId,
            is_DB: is_DB
          },
          { withCredentials: true }
        );
        
        this.localFavorite = true;
        this.recipe.favorite = true;
        this.$root.toast('Success', 'Recipe added to favorites', 'success');
      } catch (error) {
        console.error("Error adding to favorites:", error);
        this.$root.toast('Error', 'Failed to add recipe to favorites', 'danger');
      }
    }
  }
};
</script>

<style scoped>
.recipe-container {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 2rem;
  margin-bottom: 2rem;
}

.recipe-title {
  font-size: 2rem;
  color: #2c3e50;
  margin-bottom: 0.5rem;
  text-align: center;
}

.recipe-image-container {
  text-align: center;
  margin: 1rem 0;
}

.recipe-image {
  max-width: 100%;
  height: auto;
  max-height: 400px;
  object-fit: cover;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.recipe-info-bar {
  background-color: #e6f4ea;
  border-radius: 8px;
  margin: 1rem 0;
  align-items: center;
}

.recipe-section {
  margin-bottom: 2rem;
}

.section-title {
  color: #4caf50;
  font-size: 1.5rem;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #e6f4ea;
}

.ingredients-list, .instructions-list {
  padding-left: 1.5rem;
}

.ingredients-list li, .instructions-list li {
  margin-bottom: 0.5rem;
  line-height: 1.6;
}

.recipe-badges {
  text-align: center;
}

.badge {
  font-size: 0.9rem;
  padding: 0.5rem 1rem;
}

/* Add this to style the HTML instructions */
:deep(ol), :deep(ul) {
  padding-left: 1.5rem;
}

:deep(li) {
  margin-bottom: 0.5rem;
  line-height: 1.6;
}
</style>
