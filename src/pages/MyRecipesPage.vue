<template>
  <div class="container py-5">
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h1 class="mb-0">My Recipes</h1>
      <button class="btn btn-success" @click="showCreateModal = true">
        <i class="fas fa-plus me-2"></i>Create New Recipe
      </button>
    </div>

    <CreateRecipePage
      v-model:visible="showCreateModal"
      @created="onRecipeCreated"
    />

    <!-- Use the RecipePreviewList component -->
    <RecipePreviewList
      title="Your Created Recipes"
      source="my-recipes"
      emptyMessage="You haven't created any recipes yet. Start adding your own culinary creations to share with others!"
      ref="myRecipesList"
    />
  </div>
</template>

<script>
import RecipePreviewList from '../components/RecipePreviewList.vue';
import CreateRecipePage from './CreateRecipePage.vue';

export default {
  name: 'MyRecipesPage',
  components: { 
    RecipePreviewList,
    CreateRecipePage 
  },
  data() {
    return {
      store: this.$root.store,
      showCreateModal: false
    };
  },
  created() {
    // Check if user is logged in
    if (!this.store.username) {
      this.$router.push('/login');
      return;
    }
  },
  methods: {
    onRecipeCreated(newRecipeId) {
      this.showCreateModal = false;
      // Refresh the recipes list
      this.$refs.myRecipesList.updateRecipes();
      this.$router.push({
        path: `/recipe/${newRecipeId}`,
        query: { is_DB: true }
      });
    }
  }
};
</script>

<style scoped>
.container {
  min-height: calc(100vh - 56px);
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
