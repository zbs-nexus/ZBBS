<script setup lang="ts">
import '@/assets/main.css';
import { onMounted, ref } from 'vue';
import type { Schema } from '../../amplify/data/resource';
import { generateClient } from 'aws-amplify/data';

const client = generateClient<Schema>();

// create a reactive reference to the array of todos
const todos = ref<Array<Schema['Todo']["type"]>>([]);

function listTodos() {
  client.models.Todo.observeQuery().subscribe({
    next: ({ items, isSynced }) => {
      todos.value = items
     },
  }); 
}

function createTodo() {
  client.models.Todo.create({
    content: window.prompt("Todo content")
  }).then(() => {
    // After creating a new todo, update the list of todos
    listTodos();
  });
}
    
// fetch todos when the component is mounted
 onMounted(() => {
  listTodos();
});


// ...existing code...
// ランダムな色とサイズを返す関数
function getRandomCharStyle(id: string, idx: number) {
  let seed = 0;
  for (let i = 0; i < id.length; i++) seed += id.charCodeAt(i);
  seed += idx * 1000;
  function rand(min: number, max: number) {
    seed = (seed * 9301 + 49297) % 233280;
    const rnd = seed / 233280;
    return Math.floor(min + rnd * (max - min));
  }
  const fontSize = rand(16, 32) + 'px';
  const color = `hsl(${rand(0,360)}, 70%, 50%)`;
  return { color, fontSize };
}
function deleteTodo(id: string) {
  if (window.confirm('削除してもいいですか？')) {
    // ランダムな色を生成
    const randomColor = `hsl(${Math.floor(Math.random()*360)}, 70%, 85%)`;
    document.body.style.backgroundColor = randomColor;
    client.models.Todo.delete({ id }).then(() => {
      listTodos();
    });
  }
}

</script>

  


<template>
  <main>
    <h1>My todos</h1>
    <button @click="createTodo">+ new</button>
    <ul>
      <li 
        v-for="todo in todos" 
        :key="todo.id"
        @click="deleteTodo(todo.id)"
      >
        <span v-for="(char, idx) in (todo.content || '').split('')" :key="idx" :style="getRandomCharStyle(todo.id, idx)">{{ char }}</span>
      </li>
    </ul>
    <div>
      🥳 App successfully hosted. Try creating a new todo.
      <br />
      <a href="https://docs.amplify.aws/gen2/start/quickstart/nextjs-pages-router/">
        Review next steps of this tutorial.
      </a>
    </div>
  </main>
</template>
