<template>
  <div id="app-container" v-if="isReady" class="app-container">
    <el-container style="height: 100vh;">
      <el-header class="app-header">
        <h1>🎯 智能待办清单</h1>
      </el-header>
      <el-main class="app-main">
        <el-row :gutter="20">
          <el-col :span="24">
            <TodoList />
          </el-col>
        </el-row>
      </el-main>
      <el-footer class="app-footer">
        <p>© 2025 智能待办系统 - 结课项目</p>
      </el-footer>
    </el-container>
  </div>

  <!-- 加载中兜底 -->
  <div v-else class="fallback-loading">
    <div class="fallback-card">
      <div class="fallback-loading-icon">⏳</div>
      <div class="fallback-title">系统初始化中</div>
      <div class="fallback-desc">数据加载完成后自动显示</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import TodoList from './components/TodoList.vue'

// 延迟标记渲染就绪
const isReady = ref(false)

onMounted(async () => {
  // 模拟初始化延迟，确保Element Plus加载完成
  await new Promise(resolve => setTimeout(resolve, 300))
  isReady.value = true
})
</script>

<style scoped>
/* 主容器样式 */
.app-container {
  width: 100%;
  height: 100%;
  background-color: #f5f7fa;
}

/* 头部样式 */
.app-header {
  background-color: #1989fa;
  color: white;
  text-align: center;
  line-height: 60px;
  padding: 0 20px;
  margin: 0;
}

.app-header h1 {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
}

/* 主体样式 */
.app-main {
  padding: 20px;
  height: calc(100% - 120px);
  overflow-y: auto;
}

/* 底部样式 */
.app-footer {
  text-align: center;
  padding: 10px;
  color: #666;
  border-top: 1px solid #e5e5e5;
  height: 60px;
  line-height: 40px;
}

/* 加载中样式 */
.fallback-loading {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f5f7fa;
}

.fallback-card {
  padding: 40px;
  border-radius: 12px;
  background: white;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.fallback-loading-icon {
  font-size: 48px;
  color: #1989fa;
  margin-bottom: 20px;
}

.fallback-title {
  font-size: 18px;
  font-weight: 600;
  color: #333;
  margin-bottom: 10px;
}

.fallback-desc {
  font-size: 14px;
  color: #666;
  margin-bottom: 20px;
}
</style>