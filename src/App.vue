<template>
  <div>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title"/><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>
    <ul>
      <li v-for="article in articles" :key="article.id">
        <strong>{{ article.title }}</strong><br />
        {{ article.content }}<br />
        <button @click="edit(article)">Edit</button>
        <button @click="deleteArticle(article.id)">Delete</button>
      </li>
    </ul>
    <button @click="load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios'; 

export default {
  setup() {
    const form = reactive({
      id: '',
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
        const response = await axios[method](url, { title: form.title, content: form.content });

        if (form.id) {
          // Update artikel yang ada
          articles.value = articles.value.map((article) => 
            article.id === response.data.id ? response.data : article
          );
        } else {
          // Tambahkan artikel baru
          articles.value.push(response.data);
        }

        // Reset form
        resetForm();
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    function resetForm() {
      form.id = '';
      form.title = '';
      form.content = '';
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

    return { form, articles, save, edit, deleteArticle };
  }
};
</script>
