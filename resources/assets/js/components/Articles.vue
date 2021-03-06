<template>
    <div>

      <h2>Articles</h2>

      <form @submit.prevent="addOrUpdateArticle"  class="mb-3">
        <div class="form-group">
          <input type="text" class="form-control" placeholder="Title" v-model="article.title">
        </div>
        <div class="form-group">
          <textarea class="form-control" placeholder="Body" v-model="article.body"></textarea>
        </div>
        <button type="submit" class="btn btn-light btn-block">Save</button>
      </form>

      <button @click="resetForm()" class="btn btn-danger btn-block mb-2">Reset</button>

      <nav aria-label="Page navigation example">
          <ul class="pagination">
            <li :class="[{disabled: !pagination.prev_page_url}]" class="page-item">
                <a class="page-link" href="javascript:void(0);" @click="getArticles(pagination.prev_page_url)">Previous</a>
            </li>

            <li class="page-item disabled">
                <a class="page-link text-dark" href="javascript:void(0);">Page {{ pagination.current_page }} of {{ pagination.last_page }}</a>
            </li>
        
            <li :class="[{disabled: !pagination.next_page_url}]" class="page-item">
                <a class="page-link" href="javascript:void(0);" @click="getArticles(pagination.next_page_url)">Next</a>
            </li>
          </ul>
      </nav>

      <div class="card card-body mb-2" v-for="article in articles" :key="article.id">
          <h3>{{ article.title }}</h3>
          <p>{{ article.body }}</p>

          <hr>
          
          <button @click="editArticle(article)" class="btn btn-warning mb-2">Edit</button>
          <button @click="deleteArticle(article.id)" class="btn btn-danger">Delete</button>
      </div>

    </div>
</template>

<script>
export default {
  data() {
    return {
      articles: [], // collection of articles (index page)
      article: { // structure of one article for adding/editing
        id: '',
        title: '',
        body: ''
      },
      article_id: null, // id that will be sent for update route
      pagination: {}, // pagination and meta stuff here
      url: '/api/articles', // basic url for every route (difference in verb)
      urlToPage : '/api/articles?page=', // url to specific page
      edit: false, // decide if form is for add or update an article
      oldArticle: {} //if user writes some changes liveChange will trigger, but on reset need to restore values back
    };
  },
  computed: {
    // This property is created only for watching at once if title or body is changed
    articleTitleAndBody() {
        return `${this.article.title}|${this.article.body}`;
    }
  },
  watch : {
    articleTitleAndBody: function(val) {
      if(this.edit) {
        this.liveChange();
      }
    }
  },

  created() {
    this.getArticles();
  },

  methods: {
    getArticles(page_url) {
      page_url = page_url || this.url; //if page url is set, get articles for specific page, otherwise for basic (first one)
      axios.get(page_url)
        .then(res => {
          let responseLastPage = res.data.meta.last_page;
          // If user was on the last page and deleted article, and after that page count decrements, set current page to the new latest 
          if(this.pagination.current_page > responseLastPage) {
            this.getArticles(this.urlToPage + responseLastPage);
          }
          this.articles = res.data.data;
          this.makePagination(res.data.meta, res.data.links);
        })
        .catch(err => console.log(err));
    },

    makePagination(meta, links) {
      let pagination = {
        current_page: meta.current_page,
        last_page: meta.last_page,
        next_page_url: links.next,
        prev_page_url: links.prev
      };

      this.pagination = pagination;
    },

    addOrUpdateArticle() {
      this.edit ? this.updateArticle(this.article_id) : this.addArticle();
    },

    addArticle() {
        axios.post(this.url, this.article)
          .then(data => {
            this.resetForm();
            this.getArticles();
          })
          .catch(err => console.log(err));
    },

    editArticle(article) {
      this.edit = true;
      this.article_id = article.id;
      this.article = article;
    },

    updateArticle(id) {
        axios.put(`${this.url}/${id}`, this.article)
          .then(data => {
            this.resetForm();
            this.getArticles(this.urlToPage + this.pagination.current_page);
          })
          .catch(err => console.log(err));
    },

    deleteArticle(id) {
      if (confirm('Are You Sure?')) {
        axios.delete(`${this.url}/${id}`)
          .then(data => {
            this.getArticles(this.urlToPage + this.pagination.current_page);
          })
          .catch(err => console.log(err));
      }
    },

    liveChange() {
        // Iterate through all articles and find FIRST where iterable article.id === article_id setted when we click on "Edit" some article
        let thisArticle = this.articles.find(article => article.id === this.article_id);
        //Save old values in temp object, in case user press reset to restore old values
        if(!this.oldArticle.title) this.oldArticle.title = thisArticle.title;
        if(!this.oldArticle.body) this.oldArticle.body = thisArticle.body;
        //Live change
        thisArticle.title = this.article.title;
        thisArticle.body = this.article.body;
    },

    resetUnsavedChanges() {
        if(this.edit){
          // Iterate through all articles and find FIRST where iterable article.id === article_id setted when we click on "Edit" some article
          let thisArticle = this.articles.find(article => article.id === this.article_id); 
          thisArticle.title = this.oldArticle.title;
          thisArticle.body = this.oldArticle.body;
        }
        this.oldArticle = {};
    },

    resetForm() {
      this.resetUnsavedChanges();
      this.edit = false;
      this.article_id = null;
      this.article = {};
    }
  }
};
</script>