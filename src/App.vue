<script setup>
import { ref, computed, watch } from 'vue'
import AddBookForm from './components/AddBookForm.vue'
import BookFilters from './components/BookFilters.vue'
import BookCard from './components/BookCard.vue'

// Состояние книг с загрузкой из localStorage [cite: 413]
const books = ref([])
const savedBooks = localStorage.getItem('books')
if (savedBooks) {
  books.value = JSON.parse(savedBooks)
}

const currentFilter = ref('all')
const searchQuery = ref('')

// Сохранение в localStorage при изменениях [cite: 413]
watch(books, (newBooks) => {
  localStorage.setItem('books', JSON.stringify(newBooks))
}, { deep: true })

const addBook = (bookData) => {
  books.value.push({
    id: Date.now(),
    ...bookData,
    completed: false,
    rating: 0,
    isFavorite: false
  })
}

const toggleFavorite = (id) => {
  const book = books.value.find(b => b.id === id)
  if (book) {
    book.isFavorite = !book.isFavorite 
  }
}

const toggleBook = (id) => {
  const book = books.value.find(b => b.id === id)
  if (book) book.completed = !book.completed
}

const deleteBook = (id) => {
  if (confirm('Удалить книгу?')) {
    books.value = books.value.filter(b => b.id !== id)
  }
}

const rateBook = (id, rating) => {
  const book = books.value.find(b => b.id === id)
  if (book) {
    book.rating = rating
  }
}

const filteredBooks = computed(() => {
  return books.value
    .filter(book => {
      if (currentFilter.value === 'unread') return !book.completed
      if (currentFilter.value === 'read') return book.completed
      return true
    })
    .filter(book => {
      const query = searchQuery.value.toLowerCase()
      return book.title.toLowerCase().includes(query) || book.author.toLowerCase().includes(query)
    })
})
</script>

<template>
  <div class="app">
    <header>
      <h1>Менеджер книг</h1>
    </header>
    <main>
      <AddBookForm @add-book="addBook" />
      <BookFilters v-model:searchQuery="searchQuery" v-model:filter="currentFilter" :books="books" />
      <div class="books-list">
        <BookCard 
          v-for="book in filteredBooks" 
          :key="book.id" 
          :book="book" 
          @toggle="toggleBook(book.id)" 
          @delete="deleteBook(book.id)"
          @rate="(rating) => rateBook(book.id, rating)"
          @toggle-favorite="toggleFavorite(book.id)" 
        />
      </div>
    </main>
  </div>
</template>