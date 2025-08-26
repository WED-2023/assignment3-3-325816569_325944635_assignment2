<template>
  <div class="container py-5">
    <div class="d-flex justify-content-between align-items-center mb-4">
      <h1 class="mb-0">My Favorite Recipes</h1>
      <router-link to="/search" class="btn btn-primary">
        <i class="fas fa-search me-2"></i>Discover More Recipes
      </router-link>
    </div>

    <!-- Use the RecipePreviewList component -->
    <RecipePreviewList
      title="Your Saved Favorites"
      source="favorites"
      emptyMessage="You haven't saved any favorite recipes yet. Start exploring and save recipes you love to your favorites!"
    />
  </div>
</template>

<script>
import RecipePreviewList from '../components/RecipePreviewList.vue';

export default {
  name: 'FavoritesPage',
  components: {
    RecipePreviewList
  },
  data() {
    return {
      store: this.$root.store
    };
  },
  created() {
    // Check if user is logged in
    if (!this.store.username) {
      this.$router.push('/login');
      return;
    }
  }
};
</script>

<style scoped>
.container {
  min-height: calc(100vh - 56px);
}

.btn-primary {
  background-color: #4caf50;
  border-color: #4caf50;
}

.btn-primary:hover {
  background-color: #45a049;
  border-color: #45a049;
}
</style>