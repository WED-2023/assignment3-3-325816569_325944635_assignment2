<template>
  <div class="recipe-card">
    <div class="image-wrapper" @click="navigateToRecipe">
      <img
        v-if="recipe.image"
        :src="recipe.image"
        class="recipe-image"
        alt="Recipe image"
      />
      <div class="hover-overlay">
        <span>Click to view recipe details</span>
      </div>
      <div class="recipe-badges">
        <span v-if="recipe.vegan" class="badge bg-success">Vegan</span>
        <span v-if="recipe.vegetarian" class="badge bg-success">Vegetarian</span>
        <span v-if="recipe.glutenFree" class="badge bg-warning">Gluten-Free</span>
        <span v-if="isViewed" class="badge bg-info">Viewed</span>
      </div>
    </div>
    <div class="card-body">
      <div class="title-wrapper">
        <h5 class="recipe-title">{{ recipe.title }}</h5>
      </div>
      <div class="recipe-info">
        <p><i class="fas fa-clock"></i> {{ recipe.readyInMinutes }} minutes</p>
        <p>
          <i class="fas fa-heart"></i> 
          <span class="likes-count">{{ recipe.popularity || recipe.aggregateLikes || 0 }} likes</span>
        </p>
      </div>
      <button 
        @click="addToFavorites" 
        class="favorite-btn"
        :class="{ 'is-favorite': isFavorite }"
      >
        <i class="fas" :class="isFavorite ? 'fa-heart' : 'fa-heart-o'"></i>
        {{ isFavorite ? 'Saved to Favorites' : 'Add to Favorites' }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "RecipePreview",
  props: {
    recipe: {
      type: Object,
      required: true
    },
    isViewed: {
      type: Boolean,
      default: false
    },
    isFavorite: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    navigateToRecipe() {
      this.$router.push(`/recipe/${this.recipe.id}`);
    },
    addToFavorites() {
      if (!this.isFavorite) {
        this.$emit('add-to-favorites', this.recipe.id);
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

.favorite-btn:hover {
  background-color: #36a372;
}

.favorite-btn.is-favorite {
  background-color: #2c3e50;
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
