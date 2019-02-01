<template>
    <div>

      <h2>Articles</h2>

      <form @submit.prevent="addOrUpdateArticle" class="mb-3">
        <div class="form-group">
          <input type="text" class="form-control" placeholder="Title" v-model="article.title">
        </div>
        <div class="form-group">
          <textarea class="form-control" placeholder="Body" v-model="article.body"></textarea>
        </div>
        <button type="submit" class="btn btn-light btn-block">Save</button>
      </form>

      <button @click="clearForm()" class="btn btn-danger btn-block mb-2">Reset</button>

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
      articles: [],
      article: {
        id: '',
        title: '',
        body: ''
      },
      article_id: null,
      pagination: {},
      url: '/api/articles',
      urlToPage : '/api/articles?page=',
      edit: false
    };
  },

  created() {
    this.getArticles();
  },

  methods: {
    getArticles(page_url) {
      page_url = page_url || this.url;
      axios.get(page_url)
        .then(res => {
          let responseLastPage = res.data.meta.last_page;
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
            this.clearForm();
            this.getArticles();
          })
          .catch(err => console.log(err));
    },

    editArticle(article) {
      this.edit = true;
      this.article.id = article.id;
      this.article_id = article.id;
      this.article.title = article.title;
      this.article.body = article.body;
    },

    updateArticle(id) {
        axios.put(`${this.url}/${id}`, this.article)
          .then(data => {
            this.clearForm();
            this.getArticles(this.urlToPage + this.pagination.current_page);
          })
          .catch(err => console.log(err));
    },

    deleteArticle(id) {
      if (confirm('Are You Sure?')) {
        axios.delete(`${this.url}/${id}`)
          .then(data => {
            // alert('Article Removed');
            this.getArticles(this.urlToPage + this.pagination.current_page);
          })
          .catch(err => console.log(err));
      }
    },

    clearForm() {
      this.edit = false;
      this.article.id = null;
      this.article_id = null;
      this.article.title = '';
      this.article.body = '';
    }
  }
};
</script>













<!--     addOrUpdateArticle() {
      if (this.edit === false) {
        // Add
        axios.post(this.url, this.article)
          .then(data => {
            this.clearForm();
            // alert('Article Added');
            this.getArticles();
          })
          .catch(err => console.log(err));
      } else {
        // Update
        axios.put(this.url, this.article)
          .then(data => {
            this.clearForm();
            // alert('Article Updated');
            this.getArticles(this.urlToPage + this.pagination.current_page);
          })
          .catch(err => console.log(err));
      }
    } -->