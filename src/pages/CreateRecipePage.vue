<template>
  <div class="container py-4">
    <div class="row justify-content-center">
      <div class="col-md-10">
        <div class="card shadow">
          <div class="card-body p-4">
            <h1 class="card-title text-center mb-4">Create New Recipe</h1>
            
            <form @submit.prevent="submitRecipe">
              <!-- Basic Recipe Information -->
              <div class="row mb-4">
                <div class="col-md-12 mb-3">
                  <label for="title" class="form-label">Recipe Title*</label>
                  <input 
                    type="text" 
                    class="form-control" 
                    id="title" 
                    v-model="recipe.title"
                    placeholder="Enter recipe title" 
                    required
                  >
                </div>
                
                <div class="col-md-6 mb-3">
                  <label for="readyInMinutes" class="form-label">Preparation Time (minutes)*</label>
                  <input 
                    type="number" 
                    class="form-control" 
                    id="readyInMinutes" 
                    v-model="recipe.readyInMinutes"
                    placeholder="e.g. 30" 
                    min="1" 
                    required
                  >
                </div>
                
                <div class="col-md-6 mb-3">
                  <label for="servings" class="form-label">Servings*</label>
                  <input 
                    type="number" 
                    class="form-control" 
                    id="servings" 
                    v-model="recipe.servings"
                    placeholder="e.g. 4" 
                    min="1" 
                    required
                  >
                </div>
                
                <div class="col-md-12 mb-3">
                  <label for="image" class="form-label">Image URL</label>
                  <input 
                    type="url" 
                    class="form-control" 
                    id="image" 
                    v-model="recipe.image"
                    placeholder="Enter image URL" 
                  >
                  <div v-if="recipe.image" class="mt-2">
                    <img :src="recipe.image" alt="Recipe preview" class="img-thumbnail" style="max-height: 150px">
                  </div>
                </div>
              </div>
              
              <!-- Dietary Restrictions -->
              <div class="mb-4">
                <h5>Dietary Information</h5>
                <div class="form-check form-check-inline">
                  <input class="form-check-input" type="checkbox" id="vegetarian" v-model="recipe.vegetarian">
                  <label class="form-check-label" for="vegetarian">Vegetarian</label>
                </div>
                <div class="form-check form-check-inline">
                  <input class="form-check-input" type="checkbox" id="vegan" v-model="recipe.vegan">
                  <label class="form-check-label" for="vegan">Vegan</label>
                </div>
                <div class="form-check form-check-inline">
                  <input class="form-check-input" type="checkbox" id="glutenFree" v-model="recipe.glutenFree">
                  <label class="form-check-label" for="glutenFree">Gluten-Free</label>
                </div>
              </div>
              
              <!-- Ingredients -->
              <div class="mb-4">
                <h5>Ingredients*</h5>
                <div 
                  v-for="(ingredient, index) in recipe.ingredients" 
                  :key="`ingredient-${index}`"
                  class="row g-2 mb-2"
                >
                  <div class="col-md-6">
                    <input 
                      type="text" 
                      class="form-control" 
                      v-model="ingredient.name"
                      placeholder="Ingredient name" 
                      required
                    >
                  </div>
                  <div class="col-md-4">
                    <input 
                      type="text" 
                      class="form-control" 
                      v-model="ingredient.amount"
                      placeholder="Amount (e.g. 2 cups)" 
                      required
                    >
                  </div>
                  <div class="col-md-2">
                    <button 
                      type="button" 
                      class="btn btn-outline-danger w-100" 
                      @click="removeIngredient(index)"
                    >
                      <i class="fas fa-trash-alt"></i>
                    </button>
                  </div>
                </div>
                <button 
                  type="button" 
                  class="btn btn-outline-success mt-2" 
                  @click="addIngredient"
                >
                  <i class="fas fa-plus me-2"></i>Add Ingredient
                </button>
              </div>
              
              <!-- Steps -->
              <div class="mb-4">
                <h5>Preparation Steps*</h5>
                <div 
                  v-for="(step, index) in recipe.steps" 
                  :key="`step-${index}`"
                  class="row g-2 mb-2"
                >
                  <div class="col-md-1 d-flex align-items-center">
                    <span class="fw-bold">{{ index + 1 }}.</span>
                  </div>
                  <div class="col-md-9">
                    <textarea 
                      class="form-control" 
                      v-model="recipe.steps[index]"
                      placeholder="Describe this step" 
                      rows="2"
                      required
                    ></textarea>
                  </div>
                  <div class="col-md-2">
                    <button 
                      type="button" 
                      class="btn btn-outline-danger w-100 h-100" 
                      @click="removeStep(index)"
                    >
                      <i class="fas fa-trash-alt"></i>
                    </button>
                  </div>
                </div>
                <button 
                  type="button" 
                  class="btn btn-outline-success mt-2" 
                  @click="addStep"
                >
                  <i class="fas fa-plus me-2"></i>Add Step
                </button>
              </div>
              
              <!-- Form Actions -->
              <div class="d-flex justify-content-between mt-5">
                <button type="button" class="btn btn-secondary" @click="$router.go(-1)">
                  <i class="fas fa-arrow-left me-2"></i>Cancel
                </button>
                <button 
                  type="submit" 
                  class="btn btn-primary"
                  :disabled="loading || !isFormValid"
                >
                  <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
                  <i v-else class="fas fa-save me-2"></i>Save Recipe
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, inject } from 'vue';
import { useRouter } from 'vue-router';
import axios from 'axios';

const router = useRouter();
const store = inject('store');
const toast = inject('toast') || ((t, m, v) => console.log(t, m, v));

// Check if user is logged in
onMounted(() => {
  if (!store.username) {
    toast("Authentication Required", "Please log in to create recipes", "warning");
    router.push('/login');
  }
});

// Recipe model
const recipe = ref({
  title: '',
  readyInMinutes: null,
  servings: 4,
  image: '',
  vegan: false,
  vegetarian: false,
  glutenFree: false,
  ingredients: [{ name: '', amount: '' }],
  steps: ['']
});

const loading = ref(false);
const error = ref(null);

// Form validation
const isFormValid = computed(() => {
  return (
    recipe.value.title && 
    recipe.value.readyInMinutes > 0 &&
    recipe.value.servings > 0 &&
    recipe.value.ingredients.length > 0 &&
    recipe.value.ingredients.every(ing => ing.name && ing.amount) &&
    recipe.value.steps.length > 0 &&
    recipe.value.steps.every(step => step.trim() !== '')
  );
});

// Ingredient management
function addIngredient() {
  recipe.value.ingredients.push({ name: '', amount: '' });
}

function removeIngredient(index) {
  if (recipe.value.ingredients.length > 1) {
    recipe.value.ingredients.splice(index, 1);
  } else {
    toast("Cannot Remove", "Recipe must have at least one ingredient", "warning");
  }
}

// Step management
function addStep() {
  recipe.value.steps.push('');
}

function removeStep(index) {
  if (recipe.value.steps.length > 1) {
    recipe.value.steps.splice(index, 1);
  } else {
    toast("Cannot Remove", "Recipe must have at least one step", "warning");
  }
}

// Submit the recipe
async function submitRecipe() {
  if (!isFormValid.value) {
    toast("Validation Error", "Please fill out all required fields", "danger");
    return;
  }
  
  loading.value = true;
  error.value = null;
  
  try {
    const response = await axios.post(
      `${store.server_domain}/recipes/create`,
      recipe.value,
      { withCredentials: true }
    );
    
    console.log("Recipe creation response:", response.data);
    
    toast("Success", "Your recipe has been created successfully!", "success");
    
    // Navigate to the newly created recipe
    const newRecipeId = response.data.recipe_id;
    router.push({
      path: `/recipe/${newRecipeId}`,
      query: { is_DB: true }
    });
  } catch (err) {
    console.error("Error creating recipe:", err);
    error.value = err.response?.data?.message || "Failed to create recipe";
    toast("Error", error.value, "danger");
  } finally {
    loading.value = false;
  }
}
</script>

<style scoped>
.card {
  border: none;
  border-radius: 12px;
}

.form-label {
  font-weight: 500;
  color: #4caf50;
}

.btn-primary {
  background-color: #4caf50;
  border-color: #4caf50;
}

.btn-primary:hover {
  background-color: #3d9140;
  border-color: #3d9140;
}

.btn-outline-success {
  color: #4caf50;
  border-color: #4caf50;
}

.btn-outline-success:hover {
  background-color: #4caf50;
  color: white;
}
</style>
