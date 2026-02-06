<script setup lang="ts">
import '@/assets/main.css'
import { ref, onMounted, onUnmounted } from 'vue'
import type { Schema } from '../../amplify/data/resource'
import { generateClient } from 'aws-amplify/data'

/* Amplify client */
const client = generateClient<Schema>()

/* State */
const todos = ref<Array<Schema['Todo']['type']>>([])

/* Subscription reference */
let subscription: any = null

/* Fetch todos (real-time) */
function listTodos() {
  subscription = client.models.Todo.observeQuery().subscribe({
    next: ({ items }) => {
      todos.value = items
    },
    error: (err) => {
      console.error('ObserveQuery error:', err)
    }
  })
}

/* Create todo */
async function createTodo() {
  const content = window.prompt('Enter todo')
  if (!content) return

  try {
    await client.models.Todo.create({
      content
    })
  } catch (error) {
    console.error('Create failed:', error)
  }
}

/* Delete todo */
async function deleteTodo(id: string) {
  const confirmed = window.confirm('Are you sure you want to delete?')
  if (!confirmed) return

  try {
    await client.models.Todo.delete({ id })
  } catch (error) {
    console.error('Delete failed:', error)
  }
}

/* Lifecycle */
onMounted(() => {
  listTodos()
})

onUnmounted(() => {
  subscription?.unsubscribe()
})
</script>

<template>
  <main>
    <h1>My Todos</h1>

    <button @click="createTodo">➕ New Todo</button>

    <ul>
      <li v-for="todo in todos" :key="todo.id">
        {{ todo.content }}
        <button @click="deleteTodo(todo.id)">❌</button>
      </li>
    </ul>

    <p v-if="todos.length === 0">
      No todos yet. Add one 👆
    </p>
  </main>
</template>

<style scoped>
main {
  max-width: 500px;
  margin: auto;
  padding: 1rem;
}

ul {
  padding: 0;
}

li {
  list-style: none;
  display: flex;
  justify-content: space-between;
  margin: 8px 0;
}

button {
  cursor: pointer;
}
</style>
