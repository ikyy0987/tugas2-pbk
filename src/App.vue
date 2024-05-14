<template>
  <div class="container">
    <div class="header">
      <h1>Menu</h1>
      <button @click="selectedMenu = 'Todos'" :class="{ active: selectedMenu === 'Todos' }">Todos</button>
      <button @click="selectedMenu = 'Post'" :class="{ active: selectedMenu === 'Post' }">Post</button>
    </div>
    <div class="content">
      <div class="todo-app" v-if="selectedMenu === 'Todos'">
        <h1>Todo List</h1>
        <input v-model="newTask" @keyup.enter="addTask" placeholder="Tambah tugas baru" />
        <ul>
          <li v-for="task in todos" :key="task.id">
            <input type="checkbox" v-model="task.completed" @change="toggleTask(task)" />
            <span :class="{ completed: task.completed }">{{ task.title }}</span>
            <button @click="editTask(task)">Edit</button>
            <button @click="removeTask(task)">Hapus</button>
          </li>
        </ul>
      </div>
      <div class="post-app" v-else-if="selectedMenu === 'Post'">
        <h1>Postingan Pengguna</h1>
        <div class="filters">
          <label for="userSelect">Pilih Pengguna:</label>
          <select v-model="selectedUser" @change="loadPosts">
            <option v-for="user in users" :value="user.id">{{ user.name }}</option>
          </select>
        </div>
        <div v-if="posts.length > 0">
          <ul>
            <li v-for="post in posts" :key="post.id">
              <h3>{{ post.title }}</h3>
              <p>{{ post.body }}</p>
            </li>
          </ul>
        </div>
        <div v-else>
          <p>Tidak ada postingan untuk pengguna yang dipilih.</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const selectedMenu = ref('Todos');
const selectedUser = ref('');
const users = ref([]);
const posts = ref([]);
const todos = ref([]);
const newTask = ref('');

function addTask() {
  if (newTask.value.trim() === '') return;
  todos.value.push({ id: Date.now(), title: newTask.value, completed: false });
  newTask.value = '';
  saveData();
}

function toggleTask(task) {
  task.completed = !task.completed;
  saveData();
}

function removeTask(task) {
  todos.value = todos.value.filter(t => t.id !== task.id);
  saveData();
}

function editTask(task) {
  const newTitle = prompt('Edit task title', task.title);
  if (newTitle !== null) {
    task.title = newTitle;
    saveData();
  }
}

function saveData() {
  localStorage.setItem('todos', JSON.stringify(todos.value));
}

function loadData() {
  const savedTodos = localStorage.getItem('todos');
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos);
  }
}

function loadUsers() {
  fetch('https://jsonplaceholder.typicode.com/users')
    .then(response => response.json())
    .then(data => {
      users.value = data;
      if (data.length > 0) {
        selectedUser.value = data[0].id; // Memilih pengguna pertama sebagai default
        loadPosts();
      }
    })
    .catch(error => console.error('Error fetching users:', error));
}

function loadPosts() {
  if (!selectedUser.value) return;
  fetch(`https://jsonplaceholder.typicode.com/posts?userId=${selectedUser.value}`)
    .then(response => response.json())
    .then(data => {
      posts.value = data;
    })
    .catch(error => console.error('Error fetching posts:', error));
}

onMounted(() => {
  loadData();
  loadUsers();
});
</script>

<style scoped>
.container {
  background: rgb(61, 61, 61);
  width: 100%;
  min-height: 100vh;
  padding: 10px;
  margin-top: -10px;
  margin-left: -10px;
  margin-bottom: -10px;
}

.header {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

.header h1 {
  margin-right: 20px;
}

.header button {
  border: none;
  background: transparent;
  color: #fff;
  font-size: 18px;
  cursor: pointer;
  margin-right: 10px;
}

.header button.active {
  font-weight: bold;
}

.content {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.todo-app {
  width: 100%;
  max-width: 500px;
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.todo-app input {
  width: 100%;
  padding: 10px;
  margin-bottom: 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.todo-app ul {
  list-style: none;
  padding: 0;
}

.todo-app li {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.todo-app li span.completed {
  text-decoration: line-through;
  color: #888;
}

.todo-app li button {
  margin-left: 10px;
  border: none;
  background: #f44;
  color: #fff;
  cursor: pointer;
  padding: 5px 10px;
  border-radius: 4px;
}

.post-app {
  width: 100%;
  max-width: 500px;
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.post-app .filters {
  margin-bottom: 20px;
}

.post-app select {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.post-app ul {
  list-style: none;
  padding: 0;
}

.post-app li {
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.post-app li h3 {
  margin: 0 0 10px 0;
}

.post-app li p {
  margin: 0;
}
</style>

