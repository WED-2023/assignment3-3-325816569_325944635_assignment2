<template>
  <b-container>
    <h3 class="title-section">
      <strong>{{ title }}:</strong>
      <slot></slot>
    </h3>
    <div class="recipes-container">
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
    }
  },
  data() {
    return {
      recipes: []
    };
  },
  mounted() {
    this.updateRecipes();
  },
  methods: {
    async updateRecipes() {
      try {
        const response = await this.axios.get(
          "https://api.spoonacular.com/recipes/random",
          {
            params: {
              limitLicense: true,
              number: 3,
              apiKey: 'b7b147413c244375812ccb826d79cdcc'
            }
          }
        );

        const recipes = response.data.recipes.map((r) => {
          return {
            id: r.id,
            title: r.title,
            readyInMinutes: r.readyInMinutes,
            image: r.image,
            popularity: r.aggregateLikes, // This will be combined with DB likes on backend
            vegan: r.vegan,
            vegetarian: r.vegetarian,
            glutenFree: r.glutenFree
          };
        });
        this.recipes = [];
        this.recipes.push(...recipes);
      } catch (error) {
        console.log(error);
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
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  margin: 0 auto;
}

.recipe-item {
  flex: 0 0 calc(33.333% - 14px);
}
</style>
