<template>
  <form @submit.prevent="addBook">
    <label for="title">
      Title
      <input type="text" id="title" v-model.trim="newBook.title" required />
    </label>
    <label for="author">
      Author
      <input type="text" id="author" v-model.trim="newBook.author" required />
    </label>
    <label for="description">
      Description (optional)
      <input type="text" id="description" v-model.trim="newBook.descipriont" />
    </label>
    <label for="year">
      Year
      <input type="number" id="year" v-model.trim="newBook.year" required />
    </label>
    <label for="rating">
      Rating (optional)
      <input type="number" id="rating" v-model.trim="newBook.rating" />
    </label>
    <button type="submit">Submit</button>
    <button type="reset" @click="$emit('closeForm')">Close form</button>
  </form>
</template>

<script>
import { reactive } from 'vue'
import { useMutation } from '@vue/apollo-composable';

import ADD_BOOK_MUTATION from '../graphql/addbook.mutation.gql';
import ALL_BOOKS_QUERY from '../graphql/allBooks.query.gql';

export default {
  emits: ['closeForm'],
  props: {
    search: {
      type: String,
      required: true,
    }
  },
  setup(props, { emit }) {
    const newBook = reactive({
      title: '',
      author: '',
      year: '',
      rating: '',
      description: '',
    })

    const { mutate: addBook, loading, result, onDone } = useMutation(
      ADD_BOOK_MUTATION, 
      () => ({
        variables: {
          input: {
            ...newBook,
            rating: parseFloat(newBook.rating),
            year: parseInt(newBook.year),
          },
        },
        update(cache, response) {
          const sourceData = cache.readQuery({
            query: ALL_BOOKS_QUERY,
            variables: {
              search: props.search
            }
          })
          
          const data = {
            allBooks: [...sourceData.allBooks, response.data.addBook],
          }

          cache.writeQuery({
            data,
            query: ALL_BOOKS_QUERY,
            variables: {
              search: props.search
            }
          })
        },
        optimisticResponse: {
          addBook: {
            __typename: 'Book',
            id: -1,
            ...newBook
          }
        }
      })
    )

    onDone(() => emit('closeForm'))

    return {
      addBook,
      newBook,
      result,
      loading
    }
  },
}
</script>

<style scoped>
  label {
    display: block;
  }
</style>