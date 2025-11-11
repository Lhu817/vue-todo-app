<template>
  <div class="app">
    <!-- 标题区域 -->
    <header class="app-header">
      <h1 class="app-title">我的待办清单</h1>
      <p class="app-subtitle">高效管理你的每日任务</p>
    </header>

    <!-- 主要内容卡片 -->
    <main class="todo-card">
      <!-- 添加待办 -->
      <div class="add-todo">
        <input v-model="newTodo" @keyup.enter="addTodo" placeholder="今天要完成什么..." />
        <button @click="addTodo">添加任务</button>
      </div>

      <!-- 筛选按钮 -->
      <div class="filters">
        <button
          v-for="filter in filters"
          :key="filter"
          @click="currentFilter = filter"
          :class="{ active: currentFilter === filter }"
        >
          {{ filter }}
        </button>
      </div>

      <!-- 待办列表 -->
      <ul class="todo-list" v-if="filteredTodos.length > 0">
        <li v-for="todo in filteredTodos" :key="todo.id" :class="{ completed: todo.completed }">
          <input type="checkbox" v-model="todo.completed" />
          <span class="todo-text">{{ todo.text }}</span>
          <button class="delete-btn" @click="removeTodo(todo.id)">×</button>
        </li>
      </ul>

      <!-- 空状态 - 根据筛选条件显示不同提示 -->
      <div v-else class="empty-state">
        <div class="icon">📝</div>
        <p v-if="currentFilter === '全部'">还没有待办事项，添加一个开始吧！</p>
        <p v-else-if="currentFilter === '未完成'">没有未完成的事项，需要再添些什么吗？</p>
        <p v-else-if="currentFilter === '已完成'">还没有已完成的事项，尽快完成一个吧</p>
      </div>

      <!-- 统计信息 -->
      <div class="stats">
        <h3>任务统计</h3>
        <div class="stats-numbers">
          <div>
            <div>总计</div>
            <div>{{ totalTodos }}</div>
          </div>
          <div>
            <div>已完成</div>
            <div>{{ completedTodos }}</div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'

export default {
  name: 'App',
  setup() {
    const newTodo = ref('')
    const todos = ref([])
    const currentFilter = ref('全部')
    const filters = ['全部', '未完成', '已完成']

    // 添加待办
    const addTodo = () => {
      if (newTodo.value.trim()) {
        todos.value.push({
          id: Date.now(),
          text: newTodo.value,
          completed: false,
        })
        newTodo.value = ''
      }
    }

    // 删除待办
    const removeTodo = (id) => {
      todos.value = todos.value.filter((todo) => todo.id !== id)
    }

    // 计算属性
    const totalTodos = computed(() => todos.value.length)
    const completedTodos = computed(() => todos.value.filter((todo) => todo.completed).length)

    // 筛选后的待办事项
    const filteredTodos = computed(() => {
      switch (currentFilter.value) {
        case '未完成':
          return todos.value.filter((todo) => !todo.completed)
        case '已完成':
          return todos.value.filter((todo) => todo.completed)
        default:
          return todos.value
      }
    })

    // localStorage 功能
    const loadTodos = () => {
      const saved = localStorage.getItem('todos')
      return saved ? JSON.parse(saved) : []
    }

    const saveTodos = () => {
      localStorage.setItem('todos', JSON.stringify(todos.value))
    }

    onMounted(() => {
      const savedTodos = loadTodos()
      if (savedTodos.length) {
        todos.value = savedTodos
      }
    })

    watch(todos, saveTodos, { deep: true })

    return {
      newTodo,
      todos,
      currentFilter,
      filters,
      filteredTodos,
      addTodo,
      removeTodo,
      totalTodos,
      completedTodos,
    }
  },
}
</script>

<style>
/* 基础样式重置 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.app {
  max-width: 500px;
  margin: 0 auto;
  padding: 30px 20px;
  min-height: 100vh;
}

/* 标题样式 */
.app-header {
  text-align: center;
  margin-bottom: 30px;
}

.app-title {
  color: white;
  font-size: 2.5rem;
  font-weight: 300;
  margin-bottom: 10px;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
}

.app-subtitle {
  color: rgba(255, 255, 255, 0.8);
  font-size: 1rem;
  font-weight: 300;
}

/* 卡片容器 */
.todo-card {
  background: white;
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

/* 添加待办事项区域 */
.add-todo {
  display: flex;
  margin-bottom: 25px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.add-todo input {
  flex: 1;
  padding: 15px 20px;
  border: none;
  background: #f8f9fa;
  font-size: 16px;
  outline: none;
  transition: all 0.3s ease;
}

.add-todo input:focus {
  background: #fff;
  box-shadow: inset 0 0 0 2px #667eea;
}

.add-todo button {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border: none;
  padding: 0 25px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.add-todo button:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.3);
}

.stats h3 {
  margin-bottom: 10px;
  font-weight: 500;
}

.stats-numbers {
  display: flex;
  justify-content: space-around;
  font-size: 1.5rem;
  font-weight: bold;
}

/* 空状态 */
.empty-state {
  text-align: center;
  padding: 40px 20px;
  color: #888;
}

.empty-state .icon {
  font-size: 3rem;
  margin-bottom: 15px;
  opacity: 0.5;
}

.empty-state p {
  font-size: 1.1rem;
}

/* 响应式设计 */
@media (max-width: 600px) {
  .app {
    padding: 20px 15px;
  }

  .todo-card {
    padding: 20px 15px;
  }

  .app-title {
    font-size: 2rem;
  }

  .add-todo {
    flex-direction: column;
  }

  .add-todo input {
    padding: 12px 15px;
  }

  .add-todo button {
    padding: 12px;
    margin-top: 8px;
  }

  .stats-numbers {
    flex-direction: column;
    gap: 10px;
  }
}

/* 动画效果 */
.slide-fade-enter-active {
  transition: all 0.3s ease;
}
.slide-fade-leave-active {
  transition: all 0.3s cubic-bezier(1, 0.5, 0.8, 1);
}
.slide-fade-enter-from {
  transform: translateY(-10px);
  opacity: 0;
}
.slide-fade-leave-to {
  transform: translateY(10px);
  opacity: 0;
}

/* 脉冲动画 */
@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

.todo-list li:focus-within {
  animation: pulse 0.5s ease;
}
</style>
