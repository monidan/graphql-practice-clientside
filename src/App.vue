<script>
import { ref } from 'vue';
import { useQuery, useResult } from '@vue/apollo-composable';

import ALL_BOOKS_QUERY from './graphql/allBooks.query.gql';

import EditRating from './components/EditRating.vue';
import AddBook from './components/AddBook.vue';

export default {
  name: 'App',
  components: {
    EditRating,
    AddBook
  },
  setup() {
    const searchTerm = ref('');
    const activeBook = ref(null);
    const showNewBookForm = ref(false);

    const { result, loading, error } = useQuery(
      ALL_BOOKS_QUERY, 
      () => ({ search: searchTerm.value }),
      () => ({ debounce: 500, })
    );

    const books = useResult(result, [], data => data.allBooks);

    return { books, searchTerm, loading, error, activeBook, showNewBookForm };
  },
}
</script>

<template>
  <div>
    <div>
      <button v-if="!showNewBookForm" @click="showNewBookForm = true">
        Add a new book
      </button>
      <add-book
        v-if="showNewBookForm"
        :search="searchTerm"
        @closeForm="showNewBookForm = false"
      ></add-book>
    </div>

    <input type="text" v-model="searchTerm">
    <p v-if="loading">loading...</p>
    <p v-else-if="error">Something wrong!</p>

    <template v-else>
      <p v-if="activeBook">
        Update "{{ activeBook.title }}" rating:
        <EditRating 
          :initialRating="activeBook.rating"
          :bookId="activeBook.id"
          @closeForm="activeBook = null"
        />
      </p>

      <template v-else>
        <p v-for="book in books" :key="book.id">
          {{ book.title }} - {{ book.rating }}
          <button @click="activeBook = book">Edit rating</button>
        </p>
      </template>
    </template>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
