<template>
  <el-card class="todo-card" shadow="hover">
    <template #header>
      <div class="card-header">
        <span>📝 我的待办清单</span>
        <el-button type="primary" size="small" @click="addTodo">添加待办</el-button>
      </div>
    </template>

    <!-- 待办输入框 -->
    <el-form :model="formData" class="todo-form" size="small">
      <el-row :gutter="10">
        <el-col :span="14">
          <el-input
            v-model="formData.content"
            placeholder="请输入待办内容"
            @keyup.enter="addTodo"
            class="todo-input"
            clearable
          />
        </el-col>
        <el-col :span="5">
          <el-select v-model="formData.category" placeholder="选择分类" style="width: 100%">
            <el-option label="工作" value="工作" />
            <el-option label="学习" value="学习" />
            <el-option label="生活" value="生活" />
            <el-option label="其他" value="其他" />
          </el-select>
        </el-col>
        <el-col :span="4">
          <el-select v-model="formData.priority" placeholder="优先级" style="width: 100%">
            <el-option label="低" value="low" />
            <el-option label="中" value="medium" />
            <el-option label="高" value="high" />
          </el-select>
        </el-col>
      </el-row>
    </el-form>

    <!-- 筛选条件 -->
    <div class="todo-filters">
      <el-select v-model="filterCategory" placeholder="筛选分类" size="small" clearable>
        <el-option label="工作" value="工作" />
        <el-option label="学习" value="学习" />
        <el-option label="生活" value="生活" />
        <el-option label="其他" value="其他" />
      </el-select>
      <el-select v-model="filterPriority" placeholder="筛选优先级" size="small" clearable>
        <el-option label="低" value="low" />
        <el-option label="中" value="medium" />
        <el-option label="高" value="high" />
      </el-select>
      <el-select v-model="filterStatus" placeholder="筛选状态" size="small" clearable>
        <el-option label="未完成" :value="false" />
        <el-option label="已完成" :value="true" />
      </el-select>
    </div>

    <!-- 待办列表 -->
    <el-list class="todo-list" border>
      <el-list-item
        v-for="(todo, index) in filteredTodos"
        :key="todo.id"
        class="todo-item"
      >
        <div class="todo-content">
          <el-checkbox v-model="todo.done" @change="updateTodo">
            <span :class="{ done: todo.done }">
              <span class="todo-text">{{ todo.content }}</span>
              <el-tag :type="getCategoryType(todo.category)" size="small" class="category-tag">{{ todo.category }}</el-tag>
              <el-tag :type="getPriorityType(todo.priority)" size="small" class="priority-tag">{{ todo.priority === 'low' ? '低' : todo.priority === 'medium' ? '中' : '高' }}</el-tag>
            </span>
          </el-checkbox>
        </div>
        <el-button
          type="danger"
          size="small"
          icon="el-icon-delete"
          @click="deleteTodo(todo.id)"
          class="delete-btn"
        />
      </el-list-item>
      <el-list-item v-if="filteredTodos.length === 0" class="empty-tip">
        <span style="color: #999;">暂无待办，点击添加开始吧～</span>
      </el-list-item>
    </el-list>

    <!-- 统计信息 -->
    <div class="todo-stats">
      <el-row :gutter="20">
        <el-col :span="12">
          <span>已完成：{{ doneCount }} / 总数：{{ todoList.length }}</span>
        </el-col>
        <el-col :span="12" class="text-right">
          <el-button
            type="text"
            @click="clearDone"
            :disabled="doneCount === 0"
            class="clear-btn"
          >
            清空已完成
          </el-button>
        </el-col>
      </el-row>
    </div>

    <!-- 统计图表 -->
    <div class="todo-chart">
      <h3>📊 任务统计</h3>
      <el-row :gutter="20">
        <el-col :span="8">
          <div class="chart-item">
            <div class="chart-label">总任务数</div>
            <div class="chart-value">{{ todoList.length }}</div>
          </div>
        </el-col>
        <el-col :span="8">
          <div class="chart-item">
            <div class="chart-label">已完成</div>
            <div class="chart-value done-value">{{ doneCount }}</div>
          </div>
        </el-col>
        <el-col :span="8">
          <div class="chart-item">
            <div class="chart-label">完成率</div>
            <div class="chart-value">{{ completionRate }}%</div>
          </div>
        </el-col>
      </el-row>
    </div>
  </el-card>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// 表单数据
const formData = ref({
  content: '',
  category: '工作',
  priority: 'medium'
})

// 待办列表数据
const todoList = ref([])

// 筛选条件
const filterCategory = ref('')
const filterPriority = ref('')
const filterStatus = ref('')

// 生成唯一ID
const generateId = () => {
  return Date.now() + Math.random().toString(36).substr(2, 9)
}

// 获取分类标签类型
const getCategoryType = (category) => {
  const categoryMap = {
    '工作': 'primary',
    '学习': 'success',
    '生活': 'warning',
    '其他': 'info'
  }
  return categoryMap[category] || 'info'
}

// 获取优先级标签类型
const getPriorityType = (priority) => {
  const priorityMap = {
    'low': 'success',
    'medium': 'warning',
    'high': 'danger'
  }
  return priorityMap[priority] || 'warning'
}

// 计算已完成数量
const doneCount = computed(() => {
  return todoList.value.filter(todo => todo.done).length
})

// 计算完成率
const completionRate = computed(() => {
  if (todoList.value.length === 0) return 0
  return Math.round((doneCount.value / todoList.value.length) * 100)
})

// 筛选后的待办列表
const filteredTodos = computed(() => {
  return todoList.value.filter(todo => {
    let match = true
    if (filterCategory.value) {
      match = match && todo.category === filterCategory.value
    }
    if (filterPriority.value) {
      match = match && todo.priority === filterPriority.value
    }
    if (filterStatus !== '') {
      match = match && todo.done === filterStatus.value
    }
    return match
  })
})

// 从本地存储加载数据
const loadFromLocalStorage = () => {
  const stored = localStorage.getItem('todoList')
  if (stored) {
    try {
      todoList.value = JSON.parse(stored)
    } catch (e) {
      console.error('Failed to parse todo list:', e)
      todoList.value = []
    }
  } else {
    // 默认数据
    todoList.value = [
      { id: generateId(), content: '完成Vue项目部署', done: false, category: '工作', priority: 'high' },
      { id: generateId(), content: '学习Element Plus', done: true, category: '学习', priority: 'medium' },
      { id: generateId(), content: '购买生活用品', done: false, category: '生活', priority: 'low' }
    ]
    saveToLocalStorage()
  }
}

// 保存到本地存储
const saveToLocalStorage = () => {
  localStorage.setItem('todoList', JSON.stringify(todoList.value))
}

// 添加待办
const addTodo = () => {
  if (!formData.value.content.trim()) return
  const newTodo = {
    id: generateId(),
    content: formData.value.content.trim(),
    done: false,
    category: formData.value.category,
    priority: formData.value.priority
  }
  todoList.value.unshift(newTodo)
  formData.value.content = ''
  saveToLocalStorage()
}

// 更新待办
const updateTodo = () => {
  saveToLocalStorage()
}

// 删除待办
const deleteTodo = (id) => {
  todoList.value = todoList.value.filter(todo => todo.id !== id)
  saveToLocalStorage()
}

// 清空已完成
const clearDone = () => {
  todoList.value = todoList.value.filter(todo => !todo.done)
  saveToLocalStorage()
}

// 监听待办列表变化，自动保存
watch(todoList, () => {
  saveToLocalStorage()
}, { deep: true })

// 组件挂载时加载数据
onMounted(() => {
  loadFromLocalStorage()
})
</script>

<style scoped>
.todo-card {
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.todo-form {
  margin: 20px 0;
}

.todo-input {
  width: 100%;
}

.todo-filters {
  margin: 0 0 20px 0;
  display: flex;
  gap: 10px;
}

.todo-list {
  margin: 10px 0 20px 0;
  max-height: 400px;
  overflow-y: auto;
}

.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
}

.todo-content {
  flex: 1;
}

.todo-text {
  margin-right: 10px;
}

.category-tag {
  margin-right: 5px;
}

.priority-tag {
  margin-right: 5px;
}

.done {
  text-decoration: line-through;
  color: #999;
}

.delete-btn {
  opacity: 0;
  transition: opacity 0.2s;
}

.todo-item:hover .delete-btn {
  opacity: 1;
}

.empty-tip {
  text-align: center;
  padding: 20px 0;
}

.todo-stats {
  margin-top: 20px;
  padding-top: 10px;
  border-top: 1px solid #eee;
  color: #666;
}

.todo-chart {
  margin-top: 20px;
  padding: 15px;
  background-color: #fafafa;
  border-radius: 8px;
}

.todo-chart h3 {
  margin: 0 0 15px 0;
  font-size: 16px;
  font-weight: 600;
  color: #333;
}

.chart-item {
  text-align: center;
  padding: 15px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.chart-label {
  font-size: 14px;
  color: #666;
  margin-bottom: 10px;
}

.chart-value {
  font-size: 24px;
  font-weight: 600;
  color: #333;
}

.done-value {
  color: #67c23a;
}
</style>