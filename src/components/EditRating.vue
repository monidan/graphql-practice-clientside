<template>
    <input 
      type="text"
      v-model="rating"
      :disabled="loading"
      @keyup.enter="updateRating"
      @keyup.esc="$emit('closeForm')"
    >
    <p v-if="loading">Updating...</p>
    <p v-else-if="error">Something went wrong! Error: {{ error }} </p>
</template>

<script>
import { ref } from "vue"
import { useMutation } from "@vue/apollo-composable";


import UPDATE_BOOK_MUTATION from '../graphql/updateBook.mutation.gql';

export default {
  emits: ['closeForm'],
  props: {
    initialRating: {
      type: Number,
      required: true,
    },
    bookId: {
      type: String,
      required: true,
    }
  },
  setup(props, { emit }) {
    const rating = ref(props.initialRating);
    const { mutate: updateRating, onDone, loading, error } = useMutation(UPDATE_BOOK_MUTATION, () => ({
      variables: {
        id: props.bookId,
        rating: parseFloat(rating.value)
      }
    }))

    onDone(() => {
      emit('closeForm')
    })

    return { rating, updateRating, loading, error }
  }
}
</script>

