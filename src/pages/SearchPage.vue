<template>
  <div class="container">
    <b></b>
    <h1 class="title">Search Page</h1>
    <div id="form">
      <b-form @submit.prevent="onSearch" @reset.prevent="onReset">
        <b-form-group id="search" label-cols-sm="2" label="search :" label-for="search">
          <b-form-input id="search" v-model="$v.form.search.$model" ></b-form-input>
          <b-form-invalid-feedback v-if="!$v.form.search.required">
          </b-form-invalid-feedback>
        </b-form-group>

        <b-form-group id="numberOfSearch" label-cols-sm="2" label="number of results:" label-for="numberOfSearch">
          <b-form-select id="numberOfSearch" v-model="$v.form.numberOfSearch.$model" :options="numberOfSearch"
          ></b-form-select>
        </b-form-group>

        <b-form-group id="searchBy" label-cols-sm="2" label="search by:" label-for="searchBy">
          <b-form-select id="searchBy" v-model="$v.form.searchBy.$model" :options="searchBy"></b-form-select>
        </b-form-group>

        <b-button type="submit" variant="primary" style="width:200px;" class="ml-5">Search</b-button>
      </b-form>

      <li v-for="r in recipes" :key="r.id">
        <RecipePreview class="recipePreview" :recipe="r" />
      <b-alert class="c" v-if="form.submitError" variant="warning" >Search failed</b-alert>
      </li>
    </div>
  </div>
</template>

<script>
import {
  required,
  alpha,
} from "vuelidate/lib/validators";
import RecipePreview from "../components/RecipePreview.vue";
export default {
  name: "Search",
    components: {
    RecipePreview
  },
  data() {
    return {
      form: {
        numberOfSearch: "5",

      },
      numberOfSearch: [
          { value: "5", text: '5' },
          { value: "10", text: '10' },
          { value: "15", text: '15' }
        ],
      searchBy: [
          { value: "foodname", text: 'name of food' },
          { value: "recipename", text: 'name of recipe' },
        ],
    };
  },
  mounted() {
     this.getHistory(); 
  },
  validations: {
    form: {
      search: {
        required,
        alpha
      },
      numberOfSearch: {
        required   
      },
      searchBy: {
        required
      },
    }
  },
  methods: {
    async Search() {
      try {
        console.log(this.form.numberOfSearch)//number of returns
        if(this.form.searchBy=='foodname'){
          const url= this.$root.store.server_domain + "/recipes/search/" + 'food' + "/" + this.form.search;
          const response = await this.axios.get(url);
          const recipes = response.data;
          this.recipes = [];
          this.recipes.push(...recipes);
          console.log(this.recipes)
          this.$root.store.lastSearched = this.recipes
        }
        else{
          const url=this.$root.store.server_domain + "/recipes/search/" + 'recipe' + "/" + this.form.search;
          const response = await this.axios.get(url);
          const recipes = response.data.results;
          this.recipes = [];
          this.recipes.push(...recipes);
          this.recipes = this.recipes.slice(0,3)
          console.log(this.recipes)
          this.$root.store.lastSearched = this.recipes
        }
        
      } catch (err) {
        
      }
      
    },
    onSearch() {
      this.$v.form.$touch();
      if (this.$v.form.$anyError) {
        return;
      }
      this.Search();
    },
    onReset() {
      this.form = {
        search: "",
        numberOfSearch: "5",
        searchBy: "",
      };
      this.$nextTick(() => {
        this.$v.$reset();
      });
    }
  }
};
</script>
