<template>
  <div class="container py-5">
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h1 class="mb-0">My Favorite Recipes</h1>
      <router-link to="/search" class="btn btn-primary">
        <i class="fas fa-search me-2"></i>Discover More Recipes
      </router-link>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="text-center py-5">
      <div class="spinner-border text-success" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
      <p class="mt-3">Loading your favorite recipes...</p>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="alert alert-danger">
      <i class="fas fa-exclamation-circle me-2"></i>
      {{ error }}
    </div>

    <!-- Empty State -->
    <div v-else-if="recipes.length === 0" class="empty-state text-center py-5">
      <i class="fas fa-heart fa-3x mb-3 text-muted"></i>
      <h3>You haven't saved any favorite recipes yet</h3>
      <p class="text-muted mb-4">Start exploring and save recipes you love to your favorites!</p>
      <router-link to="/" class="btn btn-primary">
        Explore Recipes
      </router-link>
    </div>

    <!-- Recipes Grid -->
    <div v-else class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
      <div v-for="recipe in recipes" :key="recipe.id" class="col">
        <div class="card h-100 recipe-card">
          <div class="image-container" @click="viewRecipe(recipe.id, recipe.is_DB)">
            <img :src="recipe.image || 'https://via.placeholder.com/300x200?text=No+Image'" 
                :alt="recipe.title"
                class="card-img-top recipe-image" />
            <div class="overlay">
              <span>View Recipe</span>
            </div>
            <div class="recipe-source">
              <span :class="recipe.is_DB ? 'bg-primary' : 'bg-info'">
                {{ recipe.is_DB ? 'User Recipe' : 'Spoonacular' }}
              </span>
            </div>
          </div>
          <div class="card-body">
            <h5 class="card-title">{{ recipe.title }}</h5>
            <div class="recipe-info">
              <span><i class="fas fa-clock text-success me-1"></i> {{ recipe.readyInMinutes }} mins</span>
              <span><i class="fas fa-heart text-danger me-1"></i> {{ recipe.popularity }} likes</span>
            </div>
            <div class="recipe-badges mt-2">
              <span v-if="recipe.vegan" class="badge bg-success me-1">Vegan</span>
              <span v-if="recipe.vegetarian" class="badge bg-success me-1">Vegetarian</span>
              <span v-if="recipe.glutenFree" class="badge bg-warning me-1">Gluten-Free</span>
              <span v-if="recipe.viewed" class="badge bg-info me-1">Viewed</span>
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
  name: 'FavoritesPage',
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
    this.fetchFavorites();
  },
  methods: {
    async fetchFavorites() {
      this.loading = true;
      this.error = null;
      
      try {
        const response = await axios.get(
          `${this.store.server_domain}/users/favorites`,
          { withCredentials: true }
        );
        
        this.recipes = response.data;
        console.log('Favorite recipes:', this.recipes);
      } catch (error) {
        console.error('Error fetching favorites:', error);
        this.error = 'Failed to load your favorite recipes. Please try again.';
      } finally {
        this.loading = false;
      }
    },
    viewRecipe(id, is_DB) {
      // Navigate to the recipe view page with the is_DB query parameter
      this.$router.push({
        path: `/recipe/${id}`,
        query: { is_DB }
      });
    }
  }
};
</script>

<style scoped>
.recipe-card {
  transition: transform 0.2s;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
  border: none;
  border-radius: 10px;
  overflow: hidden;
}

.recipe-card:hover {
  transform: translateY(-5px);
}

.image-container {
  position: relative;
  overflow: hidden;
  height: 200px;
  cursor: pointer;
}

.recipe-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s;
}

.image-container:hover .recipe-image {
  transform: scale(1.05);
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: bold;
  opacity: 0;
  transition: opacity 0.3s;
}

.image-container:hover .overlay {
  opacity: 1;
}

.recipe-source {
  position: absolute;
  top: 10px;
  right: 10px;
}

.recipe-source span {
  display: inline-block;
  padding: 5px 10px;
  border-radius: 4px;
  color: white;
  font-size: 0.75rem;
  font-weight: bold;
}

.recipe-info {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
  font-size: 0.9rem;
  color: #666;
}

.card-title {
  font-size: 1.1rem;
  font-weight: bold;
  height: 2.6em;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  display: box;
  -webkit-line-clamp: 2;
  line-clamp: 2; /* Standard property for compatibility */
  -webkit-box-orient: vertical;
  box-orient: vertical;
}

.empty-state {
  background-color: white;
  border-radius: 10px;
  padding: 2rem;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.empty-state i {
  color: #e74c3c;
}
</style>
