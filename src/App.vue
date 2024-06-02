<template>
  <div>
    <form @submit.prevent="save">
      <input type="text" v-model="form.title" placeholder="Title"/><br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit">Save</button>
    </form>
    
    <q-table
      :rows="articles"
      :columns="columns"
      row-key="id"
    >
      <template v-slot:body-cell-action="props">
        <q-td :props="props">
          <q-btn flat round icon="edit" @click="edit(props.row)" />
          <q-btn flat round icon="delete" @click="deleteArticle(props.row.id)" />
        </q-td>
      </template>
    </q-table>
    
    <button @click="load">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';
import { QTable, QBtn, QTd } from 'quasar';

export default {
  components: {
    QTable,
    QBtn,
    QTd
  },
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);
    const columns = [
      { name: 'title', required: true, label: 'Title', align: 'left', field: row => row.title, format: val => `${val}`, sortable: true },
      { name: 'content', align: 'left', label: 'Content', field: row => row.content, sortable: true },
      { name: 'action', align: 'right', label: 'Actions', field: 'action', sortable: false }
    ];

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
        const url = form.id ? `http://localhost:3000/articles/${form.id}` : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (method === 'put') {
          articles.value = articles.value.map(article =>
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

    return { form, articles, save, edit, load, deleteArticle, columns };
  }
};
</script>

