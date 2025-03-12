<template>
  <div class="todo-container">
    <h2>待办事项清单（持久化版）</h2>
    
    <!-- 添加新任务的输入框 -->
    <div class="input-container">
      <input 
        v-model="newTodo" 
        @keyup.enter="addTodo"
        placeholder="请输入新的待办事项"
        type="text"
      >
      <button @click="addTodo">添加</button>
    </div>

    <!-- 进行中的任务 -->
    <div class="task-section">
      <h2>正在进行 <span class="count">({{ activeTodos.length }})</span></h2>
      <ul class="todo-list">
        <li v-for="todo in activeTodos" :key="todo.id">
          <input 
            type="checkbox" 
            v-model="todo.completed"
            @change="toggleTodo(todo)"
          >
          <span>{{ todo.text }}</span>
          <button @click="removeTodo(todo.id)" class="delete-btn">删除</button>
        </li>
      </ul>
    </div>

    <!-- 已完成的任务 -->
    <div class="task-section">
      <h2>已完成 <span class="count">({{ completedTodos.length }})</span></h2>
      <ul class="todo-list">
        <li v-for="todo in completedTodos" :key="todo.id" class="completed">
          <input 
            type="checkbox" 
            v-model="todo.completed"
            @change="toggleTodo(todo)"
          >
          <span>{{ todo.text }}</span>
          <button @click="removeTodoFromCompleted(todo.id)" class="delete-btn">删除</button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'HelloWorld',
  data() {
    return {
      newTodo: '',
      todos: [],
      api: axios.create({
        baseURL: 'http://8.134.135.150:6666/api'  // 修改端口为 6666
      })
    }
  },
  computed: {
    activeTodos() {
      return this.todos.filter(todo => !todo.completed)
    },
    completedTodos() {
      return this.todos.filter(todo => todo.completed)
    }
  },
  methods: {
    async fetchTodos() {
      try {
        const response = await this.api.get('/todos')
        this.todos = response.data
      } catch (error) {
        console.error('获取待办事项失败:', error)
      }
    },
    async addTodo() {
      if (this.newTodo.trim()) {
        try {
          const response = await this.api.post('/todos', {
            text: this.newTodo,
            completed: false
          })
          this.todos.push(response.data)
          this.newTodo = ''
        } catch (error) {
          console.error('添加待办事项失败:', error)
        }
      }
    },
    async toggleTodo(todo) {
      try {
        const response = await this.api.put(`/todos/${todo.id}`, {
          id: todo.id,
          text: todo.text,
          completed: !todo.completed,
          createTime: todo.createTime,
          updateTime: todo.updateTime
        })
        const index = this.todos.findIndex(t => t.id === todo.id)
        if (index !== -1) {
          // 保持原有状态直到收到响应
          const updatedTodo = response.data
          this.todos.splice(index, 1, updatedTodo)
          // 强制更新 completed 状态
          todo.completed = updatedTodo.completed
        }
      } catch (error) {
        console.error('更新待办事项失败:', error)
        // 如果失败，恢复复选框状态
        todo.completed = !todo.completed
      }
    },
    async removeTodo(todoId) {
      try {
        await this.api.delete(`/todos/${todoId}`)
        const index = this.todos.findIndex(todo => todo.id === todoId)
        this.todos.splice(index, 1)
      } catch (error) {
        console.error('删除待办事项失败:', error)
      }
    },
    removeTodoFromCompleted(todoId) {
      this.removeTodo(todoId)
    }
  },
  mounted() {
    this.fetchTodos()
  }
}
</script>

<style scoped>
.todo-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.input-container {
  display: flex;
  margin-bottom: 20px;
}

input[type="text"] {
  flex: 1;
  padding: 8px;
  margin-right: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

.task-section {
  margin-bottom: 30px;
}

.task-section h2 {
  font-size: 1.2em;
  color: #333;
  margin-bottom: 10px;
}

.count {
  color: #666;
  font-size: 0.9em;
}

.todo-list {
  list-style: none;
  padding: 0;
}

.todo-list li {
  display: flex;
  align-items: center;
  padding: 10px;
  margin-bottom: 5px;
  background-color: #f9f9f9;
  border-radius: 4px;
}

.todo-list li.completed span {
  text-decoration: line-through;
  color: #888;
}

.todo-list li span {
  flex: 1;
  margin: 0 10px;
}

.delete-btn {
  background-color: #f44336;
}

.delete-btn:hover {
  background-color: #da190b;
}
</style>
