<template>
  <div class="search-page">
    <div class="container py-4">
      <h1 class="text-center mb-4">Recipe Search</h1>
      
      <!-- Search Form -->
      <div class="search-form card mb-4">
        <div class="card-body">
          <div class="row g-3">
            <div class="col-md-6">
              <input v-model="searchQuery" 
                     type="text" 
                     class="form-control form-control-lg"
                     placeholder="Search recipes..."
                     @keyup.enter="performSearch" />
            </div>
            <div class="col-md-2">
              <select v-model="resultsCount" class="form-select form-select-lg">
                <option value="5">5 results</option>
                <option value="10">10 results</option>
                <option value="15">15 results</option>
              </select>
            </div>
            <div class="col-md-4">
              <button @click="performSearch" 
                      class="btn btn-success btn-lg w-100"
                      :disabled="loading">
                <i class="fas fa-search me-2"></i>Search
              </button>
            </div>
          </div>

          <!-- Filters -->
          <div class="row mt-3">
            <div class="col-md-4">
              <label class="form-label">Cuisine</label>
              <select v-model="filters.cuisine" class="form-select">
                <option value="">Any Cuisine</option>
                <option v-for="cuisine in cuisines" :key="cuisine" :value="cuisine">
                  {{ cuisine }}
                </option>
              </select>
            </div>
            <div class="col-md-4">
              <label class="form-label">Diet</label>
              <select v-model="filters.diet" class="form-select">
                <option value="">Any Diet</option>
                <option v-for="diet in diets" :key="diet" :value="diet">
                  {{ diet }}
                </option>
              </select>
            </div>
            <div class="col-md-4">
              <label class="form-label">Intolerance</label>
              <select v-model="filters.intolerance" class="form-select">
                <option value="">No Restrictions</option>
                <option v-for="intolerance in intolerances" 
                        :key="intolerance" 
                        :value="intolerance">
                  {{ intolerance }}
                </option>
              </select>
            </div>
          </div>

          <!-- Sort Options -->
          <div class="row mt-3">
            <div class="col-md-4">
              <label class="form-label">Sort By</label>
              <select v-model="sortBy" class="form-select">
                <option value="time">Preparation Time</option>
                <option value="popularity">Popularity</option>
              </select>
            </div>
          </div>
        </div>
      </div>

      <!-- Results Section -->
      <div v-if="loading" class="text-center py-5">
        <div class="spinner-border text-success" role="status">
          <span class="visually-hidden">Loading...</span>
        </div>
      </div>

      <div v-else-if="error" class="alert alert-danger">
        {{ error }}
      </div>

      <div v-else-if="recipes.length === 0 && searched" 
           class="alert alert-info text-center">
        No recipes found. Try different search terms or filters.
      </div>

      <div v-else class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
        <div v-for="recipe in sortedRecipes" :key="recipe.id" class="col">
          <div class="card h-100 recipe-card">
            <img :src="recipe.image" 
                 :alt="recipe.title"
                 class="card-img-top recipe-image"
                 @click="viewRecipe(recipe.id)" />
            <div class="card-body">
              <h5 class="card-title">{{ recipe.title }}</h5>
              <p class="card-text">
                <i class="far fa-clock me-2"></i>
                {{ recipe.readyInMinutes }} minutes
              </p>
              <p class="card-text">
                <i class="far fa-heart me-2"></i>
                {{ recipe.aggregateLikes }} likes
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, inject, onBeforeUnmount } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';
import { CUISINES, DIETS, INTOLERANCES } from '../utils/recipeConstants';

const router = useRouter();
const store = inject('store');
const searchQuery = ref(localStorage.getItem('lastSearch') || '');
const resultsCount = ref(5);
const loading = ref(false);
const error = ref('');
const recipes = ref([]);
const searched = ref(false);
const sortBy = ref('time');

const filters = ref({
  cuisine: '',
  diet: '',
  intolerance: ''
});

// Using imported constants
const cuisines = CUISINES;
const diets = DIETS;
const intolerances = INTOLERANCES;

const sortedRecipes = computed(() => {
  return [...recipes.value].sort((a, b) => {
    if (sortBy.value === 'time') {
      return a.readyInMinutes - b.readyInMinutes;
    }
    return b.aggregateLikes - a.aggregateLikes;
  });
});

async function performSearch() {
  if (!searchQuery.value) return;
  
  loading.value = true;
  error.value = '';
  searched.value = true;
  
  try {
    localStorage.setItem('lastSearch', searchQuery.value);
    
    // Use GET with query parameters instead of POST with body
    const response = await axios.get(`${store.server_domain}/recipes/search`, {
      params: {
        query: searchQuery.value,
        number: resultsCount.value,
        cuisine: filters.value.cuisine,
        diet: filters.value.diet,
        intolerances: filters.value.intolerance,
        addRecipeInformation: true,
        instructionsRequired: true
      }
    });
    
    console.log('Search response:', response.data);
    
    // Handle the response data correctly
    recipes.value = Array.isArray(response.data) ? response.data : [];
  } catch (e) {
    error.value = 'Failed to fetch recipes. Please try again.';
    console.error('Search error:', e);
  } finally {
    loading.value = false;
  }
}

function viewRecipe(id) {
  router.push(`/recipe/${id}`);
}

onMounted(() => {
  if (searchQuery.value) {
    performSearch();
  }
});

// Clear search state when leaving the page
onBeforeUnmount(() => {
  searchQuery.value = '';
  resultsCount.value = 5;
  recipes.value = [];
  error.value = '';
  searched.value = false;
  sortBy.value = 'time';
  filters.value.cuisine = '';
  filters.value.diet = '';
  filters.value.intolerance = '';
  localStorage.removeItem('lastSearch');
});
</script>

<style scoped>
.search-page {
  background-color: #e6f4ea;
  min-height: calc(100vh - 56px);
}

.search-form {
  background-color: white;
  border: none;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.recipe-card {
  transition: transform 0.2s;
  cursor: pointer;
}

.recipe-card:hover {
  transform: translateY(-5px);
}

.recipe-image {
  height: 200px;
  object-fit: cover;
}

.form-label {
  color: #4caf50;
  font-weight: 500;
}

.btn-success {
  background-color: #4caf50;
  border-color: #4caf50;
}

.btn-success:hover {
  background-color: #45a049;
  border-color: #45a049;
}
</style>
