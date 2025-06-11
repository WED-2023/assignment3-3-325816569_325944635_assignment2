<template>
  <div class="container py-4">
    <h1 class="text-center mb-5">Family Recipes</h1>
    
    <!-- Loading State -->
    <div v-if="loading" class="text-center py-5">
      <div class="spinner-border text-success" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      <p class="mt-3">Loading family recipes...</p>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="alert alert-danger">
      <i class="fas fa-exclamation-circle me-2"></i>
      {{ error }}
    </div>

    <!-- Empty State -->
    <div v-else-if="recipes.length === 0" class="text-center py-5">
      <div class="empty-state">
        <i class="fas fa-users fa-3x mb-3 text-muted"></i>
        <h3>No Family Recipes Found</h3>
        <p class="text-muted mb-4">This is where your family's traditional recipes would appear.</p>
      </div>
    </div>

    <!-- Recipes Display -->
    <div v-else>
      <div class="row g-4">
        <div v-for="recipe in recipes" :key="recipe.id" class="col-md-6 col-lg-4">
          <div class="card family-recipe-card h-100">
            <div class="family-member-badge">
              <i class="fas fa-user me-1"></i> {{ recipe.family_member }}
            </div>
            
            <img 
              :src="recipe.image || 'https://via.placeholder.com/300x200?text=No+Image'" 
              class="card-img-top family-recipe-image" 
              :alt="recipe.title"
            />
            
            <div class="card-body">
              <h5 class="card-title">{{ recipe.title }}</h5>
              
              <div class="recipe-meta">
                <span><i class="fas fa-clock text-success me-1"></i> {{ recipe.readyInMinutes }} mins</span>
                <span><i class="fas fa-utensils text-success me-1"></i> {{ recipe.servings }} servings</span>
              </div>
              
              <div class="recipe-badges mt-2 mb-3">
                <span v-if="recipe.vegan" class="badge bg-success me-1">Vegan</span>
                <span v-if="recipe.vegetarian" class="badge bg-success me-1">Vegetarian</span>
                <span v-if="recipe.glutenFree" class="badge bg-warning me-1">Gluten-Free</span>
              </div>
              
              <div class="accordion" :id="`accordion-${recipe.id}`">
                <div class="accordion-item">
                  <h2 class="accordion-header" :id="`ingredients-heading-${recipe.id}`">
                    <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" 
                      :data-bs-target="`#ingredients-${recipe.id}`" aria-expanded="false" 
                      :aria-controls="`ingredients-${recipe.id}`">
                      Ingredients
                    </button>
                  </h2>
                  <div :id="`ingredients-${recipe.id}`" class="accordion-collapse collapse" 
                    :aria-labelledby="`ingredients-heading-${recipe.id}`" 
                    :data-bs-parent="`#accordion-${recipe.id}`">
                    <div class="accordion-body">
                      <ul class="ingredients-list">
                        <li v-for="(ingredient, idx) in recipe.ingredients" :key="idx">
                          {{ ingredient.name }} ({{ ingredient.amount }})
                        </li>
                      </ul>
                    </div>
                  </div>
                </div>
                
                <div class="accordion-item">
                  <h2 class="accordion-header" :id="`steps-heading-${recipe.id}`">
                    <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse" 
                      :data-bs-target="`#steps-${recipe.id}`" aria-expanded="false" 
                      :aria-controls="`steps-${recipe.id}`">
                      Instructions
                    </button>
                  </h2>
                  <div :id="`steps-${recipe.id}`" class="accordion-collapse collapse" 
                    :aria-labelledby="`steps-heading-${recipe.id}`" 
                    :data-bs-parent="`#accordion-${recipe.id}`">
                    <div class="accordion-body">
                      <ol class="steps-list">
                        <li v-for="(step, idx) in recipe.steps" :key="idx">
                          {{ step }}
                        </li>
                      </ol>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'FamilyRecipesPage',
  data() {
    return {
      recipes: [],
      loading: true,
      error: null,
      store: this.$root.store
    };
  },
  created() {
    // Check if user is logged in
    if (!this.store.username) {
      this.$router.push('/login');
      return;
    }
    this.fetchFamilyRecipes();
  },
  methods: {
    async fetchFamilyRecipes() {
      this.loading = true;
      this.error = null;
      
      try {
        const response = await axios.get(
          `${this.store.server_domain}/users/family-recipes`,
          { withCredentials: true }
        );
        
        console.log('Family recipes:', response.data);
        this.recipes = response.data;
      } catch (error) {
        console.error('Error fetching family recipes:', error);
        this.error = 'Failed to load family recipes. Please try again.';
      } finally {
        this.loading = false;
      }
    },
    
    viewFullRecipe(recipe) {
      // Store the full recipe details temporarily
      this.$root.fullRecipeDetails = recipe;
      
      // Navigate to a dedicated view with family recipe context
      this.$router.push({
        path: `/recipe/${recipe.id}`,
        query: { 
          is_DB: true,
          family: true 
        }
      });
    }
  }
};
</script>

<style scoped>
.family-recipe-card {
  position: relative;
  overflow: hidden;
  border: none;
  border-radius: 12px;
  box-shadow: 0 2px 15px rgba(0,0,0,0.1);
  transition: transform 0.3s;
}

.family-recipe-card:hover {
  transform: translateY(-5px);
}

.family-recipe-image {
  height: 200px;
  object-fit: cover;
}

.family-member-badge {
  position: absolute;
  top: 15px;
  right: 15px;
  background-color: rgba(255, 255, 255, 0.9);
  color: #4caf50;
  padding: 8px 12px;
  border-radius: 20px;
  font-weight: bold;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  z-index: 1;
}

.recipe-meta {
  display: flex;
  justify-content: space-between;
  margin: 10px 0;
  font-size: 0.9rem;
  color: #666;
}

.accordion-button:not(.collapsed) {
  background-color: #e6f4ea;
  color: #4caf50;
}

.accordion-button:focus {
  box-shadow: 0 0 0 0.25rem rgba(76, 175, 80, 0.25);
}

.ingredients-list, .steps-list {
  padding-left: 1.5rem;
  margin-bottom: 0;
}

.ingredients-list li, .steps-list li {
  margin-bottom: 0.5rem;
}

.empty-state {
  background-color: white;
  border-radius: 10px;
  padding: 2rem;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.empty-state i {
  color: #4caf50;
}
</style>
