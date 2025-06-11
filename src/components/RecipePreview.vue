<template>
  <div class="recipe-card">
    <div class="image-wrapper" @click="navigateToRecipe">
      <img
        v-if="localRecipe.image"
        :src="localRecipe.image"
        class="recipe-image"
        alt="Recipe image"
      />
      <div class="hover-overlay">
        <span>Click to view recipe details</span>
      </div>
      <div class="recipe-badges">
        <span v-if="localRecipe.vegan" class="badge bg-success">Vegan</span>
        <span v-if="localRecipe.vegetarian" class="badge bg-success">Vegetarian</span>
        <span v-if="localRecipe.glutenFree" class="badge bg-warning">Gluten-Free</span>
        <span v-if="localRecipe.viewed" class="badge bg-info">Viewed</span>
        <span v-else-if="store.username" class="badge bg-secondary">Not Viewed</span>
      </div>
    </div>
    <div class="card-body">
      <div class="title-wrapper">
        <h5 class="recipe-title">{{ localRecipe.title }}</h5>
      </div>
      <div class="recipe-info">
        <p><i class="fas fa-clock"></i> {{ localRecipe.readyInMinutes }} minutes</p>
        <p>
          <i class="fas fa-heart"></i> 
          <span class="likes-count">{{ localRecipe.popularity || localRecipe.aggregateLikes || 0 }} likes</span>
        </p>
      </div>
      <button 
        @click="addToFavorites" 
        class="favorite-btn"
        :class="{ 'is-favorite': localRecipe.favorite }"
        :disabled="localRecipe.favorite"
      >
        <i class="fas" :class="localRecipe.favorite ? 'fa-heart' : 'fa-heart-o'"></i>
        {{ localRecipe.favorite ? 'Saved to Favorites' : 'Add to Favorites' }}
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: "RecipePreview",
  props: {
    recipe: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      store: this.$root.store,
      localRecipe: { ...this.recipe } // Create a local copy of the prop
    };
  },
  watch: {
    // Update local copy when prop changes
    recipe: {
      handler(newVal) {
        this.localRecipe = { ...newVal };
        console.log("Recipe updated, favorite status:", this.localRecipe.favorite);
      },
      deep: true,
      immediate: true
    }
  },
  mounted() {
    // Check the favorite status when component mounts if user is logged in
    if (this.store.username && !this.localRecipe.favorite) {
      this.checkFavoriteStatus();
    }
  },
  methods: {
    async checkFavoriteStatus() {
      try {
        const recipeId = this.localRecipe.id;
        const is_DB = this.localRecipe.is_DB;
        
        const response = await axios.get(
          `${this.$root.store.server_domain}/users/favorites/${recipeId}`,
          {
            params: { is_DB },
            withCredentials: true
          }
        );
        
        if (response.data.isFavorite) {
          this.localRecipe.favorite = true;
        }
      } catch (error) {
        console.error("Error checking favorite status:", error);
      }
    },
    navigateToRecipe() {
      // Pass is_DB as a query parameter instead of changing the path
      this.$router.push({
        path: `/recipe/${this.localRecipe.id}`,
        query: { is_DB: this.localRecipe.is_DB }
      });
    },
    async addToFavorites() {
      if (!this.store.username) {
        // If user is not logged in, redirect to login
        this.$router.push('/login');
        return;
      }
      
      // Don't allow adding if already a favorite
      if (this.localRecipe.favorite) {
        return;
      }
      
      try {
        // Add detailed logging to debug the issue
        console.log("Recipe object:", this.localRecipe);
        
        // Handle case where is_DB might be undefined
        let is_DB = this.localRecipe.is_DB;
        
        const recipeId = this.localRecipe.id;
        
        console.log("Adding to favorites with params:", {
          recipeId,
          is_DB,
          url: `${this.$root.store.server_domain}/users/favorites`
        });
        
        // Add to favorites - INCLUDE CREDENTIALS
        const response = await axios.post(`${this.$root.store.server_domain}/users/favorites`, {
          recipeId: recipeId,
          is_DB: is_DB
        }, {
          withCredentials: true // This is critical for sending cookies with the request
        });
        
        console.log("Server response:", response);
        
        // Update local state
        this.localRecipe.favorite = true;
        this.$root.toast('Success', 'Recipe added to favorites', 'success');
      } catch (error) {
        console.error('Error adding to favorites:', error);
        
        // More detailed error logging
        if (error.response) {
          console.error('Response data:', error.response.data);
          console.error('Response status:', error.response.status);
          console.error('Response headers:', error.response.headers);
        } else if (error.request) {
          console.error('No response received:', error.request);
        } else {
          console.error('Error message:', error.message);
        }
        
        this.$root.toast('Error', `Failed to add recipe to favorites: ${error.response?.data?.message || error.message}`, 'danger');
      }
    }
  }
}
</script>

<style scoped>
.recipe-card {
  position: relative;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: transform 0.2s;
  background: white;
  height: 100%;
}

.image-wrapper {
  position: relative;
  width: 100%;
  padding-top: 75%;
  cursor: pointer;
}

.recipe-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.hover-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  opacity: 0;
  transition: opacity 0.3s;
}

.image-wrapper:hover .hover-overlay {
  opacity: 1;
}

.recipe-badges {
  position: absolute;
  top: 10px;
  left: 10px;
  display: flex;
  gap: 5px;
  flex-wrap: wrap;
  max-width: 90%;
}

.badge {
  padding: 5px 10px;
  font-size: 0.75rem;
}

.card-body {
  padding: 1rem;
}

.recipe-title {
  font-size: 1.1rem;
  margin-bottom: 1rem;
  color: #2c3e50;
  height: 2.8em;
  position: relative;
  overflow: hidden;
  cursor: default;
}

.recipe-title:hover {
  overflow: visible;
  white-space: normal;
  height: auto;
  z-index: 1;
  background: white;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  padding: 5px;
  border-radius: 4px;
}

.recipe-title:hover + .recipe-info {
  visibility: hidden;
}

/* Add a wrapper to prevent layout shift */
.title-wrapper {
  height: 2.8em;
  position: relative;
  margin-bottom: 1rem;
}

.recipe-info {
  color: #666;
  font-size: 0.9rem;
  margin-bottom: 1rem;
}

.recipe-info i {
  margin-right: 5px;
  color: #42b983;
}

.favorite-btn {
  width: 100%;
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  background-color: #42b983;
  color: white;
  font-size: 0.9rem;
  cursor: pointer;
  transition: background-color 0.3s;
}

.favorite-btn:hover:not(:disabled) {
  background-color: #36a372;
}

.favorite-btn.is-favorite {
  background-color: #2c3e50;
  cursor: not-allowed;
}

.favorite-btn.is-favorite .fas {
  color: #e74c3c;
}

.favorite-btn i {
  margin-right: 5px;
}

.likes-count {
  font-weight: 500;
  color: #2c3e50;
}

.recipe-info p {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-bottom: 0.5rem;
}

.recipe-info i.fa-heart {
  color: #e74c3c;
}
</style>
