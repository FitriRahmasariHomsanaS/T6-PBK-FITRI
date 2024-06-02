<template>
  <div class="app-background">
    <div class="container">
      <h1 class="title">List Novel</h1>
      <form @submit.prevent="save" class="form">
        <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
        <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
        <button @click="save" @touchstart.prevent="save" class="button save">Save</button>
      </form>
      <ul class="article-list">
        <li v-for="article in articles" :key="article.id" class="article-item">
          <strong>{{ article.title }}</strong><br />
          <span>{{ article.content }}</span><br />
          <button @click="edit(article)" @touchstart.prevent="edit(article)" class="button edit">Edit</button>
          <button @click="deleteArticle(article.id)" @touchstart.prevent="deleteArticle(article.id)" class="button delete">Delete</button>
        </li>
      </ul>
      <button @click="load" @touchstart.prevent="load" class="button load">Load</button>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, {
          title: form.title,
          content: form.content,
        });

        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, load, deleteArticle };
  }
};
</script>

<style scoped>
.app-background {
  background-color: #8E3E63;
  padding: 20px;
}

.container {
  margin: 0 auto;
  max-width: 800px;
  padding: 20px;
  background-color: #E1ACAC;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.title {
  text-align: center;
  font-size: 2em;
  margin-bottom: 20px;
  color: #333;
}

.form {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.input, .textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.input:focus, .textarea:focus {
  border-color: #333;
}

.button {
  padding: 10px 20px;
  margin: 5px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.button.save {
  background-color: #888888;
  color: white;
}

.button.save:hover {
  background-color: #777777;
}

.button.edit, .button.delete, .button.load {
  background-color: #8E3E63;
  color: white;
}

.button.edit:hover, .button.delete:hover, .button.load:hover {
  background-color: #7d3756;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  padding: 15px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-bottom: 10px;
  background-color: #f9f9f9;
}

.article-item strong {
  font-size: 1.2em;
  color: #555555;
}
</style>
