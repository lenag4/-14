<template>
  <div class="container mt-5">
    <h1 class="text-center mb-4">Моя коллекция книг</h1>

    <div class="row mb-4">
      <div class="col-md-6">
        <div class="alert alert-info">
          Всего книг: <strong>{{ books.length }}</strong>
        </div>
      </div>
      <div class="col-md-6">
        <div class="d-flex justify-content-end">
          <select v-model="sortOrder" class="form-select w-auto">
            <option value="default">По умолчанию</option>
            <option value="asc">По названию (А-Я)</option>
            <option value="desc">По названию (Я-А)</option>
          </select>
        </div>
      </div>
    </div>
    
    <div class="row">
      <div class="col-md-8 offset-md-2">
        <form @submit.prevent="addBook" class="mb-4 p-4 border rounded bg-light">
          <h5 class="mb-3">Добавить новую книгу</h5>

          <div class="mb-3">
            <label class="form-label">Название книги</label>
            <input 
              v-model="newBook.title" 
              type="text" 
              class="form-control" 
              placeholder="Введите название"
              required
            >
          </div>

          <div class="mb-3">
            <label class="form-label">Описание</label>
            <textarea
              v-model="newBook.description"
              class="form-control"
              rows="3"
              placeholder="Введите описание книги"
            ></textarea>
          </div>

          <div class="mb-3">
            <label class="form-label">Фото книги</label>
            <input
              type="file"
              class="form-control"
              accept="image/*"
              @change="handleFileUpload"
              ref="fileInput"
            >
            <small class="text-muted">Поддерживаются форматы: JPG, PNG, GIF</small>
          </div>

          <button class="btn btn-primary" type="submit">Добавить книгу</button>
        </form>
      </div>
    </div>

    <div class="row">
      <div 
        v-for="book in sortedBooks" 
        :key="book.id" 
        class="col-md-4 mb-3"
      >
        <div class="card h-100">
          <div v-if="book.photo" class="card-img-top-wrapper" style="height: 500px; overflow: hidden;">
            <img
              :src="book.photo"
              class="card-img-top"
              :alt="book.title"
              style="width: 100%; height: 100%; object-fit: cover;"
            >
          </div>
          <div v-else class="card-img-top bg-light d-flex align-items-center justify-content-center" style="height: 200px;">
            <span class="text-muted">Нет фото</span>
          </div>

          <div class="card-body">
            <h5 class="card-title">{{ book.title }}</h5>
            <p class="card-text">{{ book.description || 'Нет описания' }}</p>
          </div>
          <div class="card-footer">
            <button 
              @click="removeBook(book.id)" 
              class="btn btn-sm btn-danger"
            >
              Удалить
            </button>
          </div>
        </div>
      </div>
    </div>
    <div v-if="books.length === 0" class="text-center py-5">
      <p class="text-muted">Коллекция пуста. Добавьте первую книгу!</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const books = ref([])
const sortOrder = ref('default')
const fileInput = ref(null)

const newBook = ref({
  title: '',
  description: '',
  photo: null
})

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  if (file) {
    if (file.size > 5 * 1024 * 1024) {
      alert('Файл слишком большой. Максимальный размер 5MB')
      clearImage()
      return
    }

    if (!file.type.startsWith('image/')) {
      alert('Пожалуйста, выберите изображение')
      clearImage()
      return
    }

    const reader = new FileReader()
    reader.onload = (e) => {
      newBook.value.photo = e.target.result
    }
    reader.readAsDataURL(file)
  }
}

const clearImage = () => {
  newBook.value.photo = null
  if (fileInput.value) {
    fileInput.value.value = ''
  }
}

const sortedBooks = computed(() => {
  let sorted = [...books.value]
  if (sortOrder.value === 'asc') {
    sorted.sort((a, b) => a.title.localeCompare(b.title))
  } else if (sortOrder.value === 'desc') {
    sorted.sort((a, b) => b.title.localeCompare(a.title))
  }
  return sorted
})

const addBook = () => {
  if (!newBook.value.title.trim()) return
  books.value.push({
    id: Date.now(),
    title: newBook.value.title,
    description: newBook.value.description || null,
    photo: newBook.value.photo
  })
  
  newBook.value = {
    title: '',
    description: '',
    photo: null
  }
  clearImage()
}

const removeBook = (id) => {
  books.value = books.value.filter(book => book.id !== id)
}
</script>