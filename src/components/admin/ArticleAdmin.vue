<template>
<div class="article-admin">
    <input id="article-admin" type="hidden" v-model="article.id">
    <b-form>  
            <b-form-group 
                label="Nome:" label-for="article-name" >
            <b-form-input 
            id="article-name" 
            type="text" 
            v-model="article.name" 
            :readonly="mode === 'remove'"
             required placeholder="Informe o Nome da Artigo."/>
          </b-form-group>
          <b-form-group 
                label="Descrição:" label-for="article-description" >
            <b-form-input 
            id="article-description" 
            type="text" 
            v-model="article.description" 
            :readonly="mode === 'remove'"
             required placeholder="Informe a descrição da Artigo."/>
          </b-form-group>             
          <b-form-group v-if="mode === 'save'"
                label="Imagem (URL):" label-for="article-imageUrl" >
            <b-form-input 
            id="article-imageUrl" 
            type="text" 
            v-model="article.imageUrl" 
            :readonly="mode === 'remove'"
            required placeholder="Informe a URL da Imagem do Artigo."/>
          </b-form-group>
           <b-form-group v-if="mode === 'save'"
              id="article-categoryId" 
               label="Categoria:" label-for="article-categoryId" >
            <b-form-select 
              v-model="article.categoryId" 
                :options="categories" />
          </b-form-group>
          <b-form-group v-if="mode === 'save'"
              id="article-userId" 
               label="Autor:" label-for="article-userId" >
            <b-form-select 
              v-model="article.userId" 
                :options="users" />
          </b-form-group>
          <b-form-group v-if="mode === 'save'"
           label="Conteúdo" label-for="article-content" >
            <VueEditor v-model="article.content" 
            placeholder="Informe o conteúdo do artigo..."/>
          </b-form-group>   
          <b-button variant="primary" v-if="mode === 'save'" @click="save">Salvar</b-button>
          <b-button variant="danger" v-if="mode === 'remove'" @click="remove">Exluir</b-button> 
          <b-button class="ml-2" @click="reset">Cancelar</b-button>         
    </b-form>
    <hr>
    <b-table hover striped :items="articles" :fields="fields">
      <template slot="actions" slot-scope="data">
        <b-button variant="danger" @click="loadArticle(data.item)" class="mr-2">
          <i class="fa fa-pencil"></i>
        </b-button>
       <button variant="danger" @click="loadArticle(data.item, 'remove')">
         <i class="fa fa-trash"></i>
       </button>
      </template>
    </b-table>
    <b-pagination size="md" v-model="page" :total-rows="count" :per-page="limit" />
</div>    
</template>

<script>
import { VueEditor } from "vue2-editor";
import { baseApiUrl, showError } from "@/global";
import axios from "axios";

export default {
  name: "ArticleAdmin",
  components: { VueEditor },
  data: function() {
    return {
      mode: "save",
      article: {},
      articles: [],
      categories: [],
      page: 1,
      limit: 0,
      count: 0,
      users: [],
      fields: [
        { key: "id", label: "Código", sortable: true },
        { key: "name", label: "Nome", sortable: true },
        { key: "descriptin", label: "Descrição", sortable: true },
        { key: "actions", label: "Ações" }
      ]
    };
  },
  methods: {
    loadArticles() {
      const url = `${baseApiUrl}/articles?page=${this.page}`;
      axios.get(url).then(res => {
          this.articles = res.data.data
          this.count = res.data.count
          this.limit = res.data.limitPaginacao
        });
    },
    reset() {
      (this.mode = "save"), (this.article = {}), this.loadArticles();
    },
    save() {
      const method = this.article.id ? "put" : "post";
      const id = this.article.id ? `/${this.article.id}` : "";
      axios[method](`${baseApiUrl}/articles${id}`, this.article)
        .then(() => {
          this.$toasted.global.defaultSuccess();
          this.reset();
        })
        .catch(showError);
    },
    remove() {
      const id = this.article.id;
      axios
        .delete(`${baseApiUrl}/articles/${id}`)
        .then(() => {
          this.$toasted.global.defaultSuccess();
          this.reset();
        })
        .catch(showError);
    },
    loadArticle(article, mode = "save") {
      this.mode = mode;
      axios.get(`${baseApiUrl}/articles/${article.id}`)
        .then(res => this.article = res.data)
    },
    loadCategories() {
        const url = `${baseApiUrl}/categories`
        axios.get(url).then(res => {
            this.categories = res.data.map(category => {
                return {value: category.id, text: category.path}
            })
        })
    },
    loadUsers() {
        const url = `${baseApiUrl}/users`
        axios.get(url).then( res => {
            this.users = res.data.map(user => {
                return {value: user.id, text: `${user.name} - ${user.email}`}
            })
        })
    }
  },
  watch: {
      page() {
          this.loadArticles();
      }
  },
  mounted() {
    this.loadArticles();
    this.loadCategories();
    this.loadUsers();
  }
};
</script>

<style>
</style>
