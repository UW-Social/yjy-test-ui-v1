<template>
  <div class="events-page">
    <!-- 背景容器 -->
    <div class="background-container">
      <img src="/svg/eventsbg.svg" alt="Clubs Background" class="background-svg" />
    </div>

    <div class="overlapping-page">
      <div class="events-page-with-search">
        <div class="events-content">
          <!-- 搜索框和排序容器 -->
          <div class="search-bar-container">
            <input
              v-model="searchQuery"
              type="text"
              placeholder="Search clubs..."
              class="search-bar"
              @keyup.enter="handleSearch"
            />
            
            <!-- 排序选择器 -->
            <div class="sort-selector">
              <label class="sort-label">Sort by:</label>
              <select v-model="sortType" class="sort-dropdown">
                <option value="recommended">Recommended</option>
                <option value="members">Members</option>
              </select>
            </div>
          </div>

          <!-- 俱乐部列表 -->
          <el-main class="event-list-container">
            <ClubList :category="categoryFilter" :search="searchQuery" :sort="sortType" @open-card="setSelectedClub" />
          </el-main>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import ClubList from '../components/ClubList.vue';
import '../assets/sidebar.css';
import { type Club } from '../types/club';

const router = useRouter();
const route = useRoute();

const categoryFilter = ref('');
const searchQuery = ref('');
const sortType = ref('recommended'); // 默认为推荐排序
const isUpdatingFromUrl = ref(false); // 标记是否正在从 URL 更新

// 从 URL 查询参数初始化搜索
onMounted(() => {
  const q = route.query.q as string;
  if (q) {
    isUpdatingFromUrl.value = true;
    searchQuery.value = q;
    isUpdatingFromUrl.value = false;
  }
});

// 监听路由查询参数变化
watch(() => route.query.q, (newQuery) => {
  isUpdatingFromUrl.value = true;
  if (newQuery) {
    searchQuery.value = newQuery as string;
  } else {
    searchQuery.value = ''; // 清空搜索
  }
  isUpdatingFromUrl.value = false;
});

// 监听搜索框变化（用户手动输入/清空）
watch(searchQuery, (newVal) => {
  // 如果是从 URL 更新的，不要再更新 URL（避免循环）
  if (isUpdatingFromUrl.value) return;
  
  if (!newVal || newVal.trim() === '') {
    // 如果搜索框为空，清除 URL 参数
    if (route.query.q) {
      router.replace({ path: route.path, query: {} });
    }
  }
});

const setSelectedClub = (club: Club) => {
  // 跳转到详情页
  router.push(`/clubs/${club.id}`);
};

const handleSearch = () => {
  console.log('Search initiated for:', searchQuery.value);
};
</script>

<style scoped>
/* Align overall layout to Events page */
.events-page {
  position: relative;
  overflow: hidden;
}

.background-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}

.background-svg {
  width: 100%;
  height: auto;
}

.overlapping-page {
  position: relative;
  z-index: 1;
}

.events-page-with-search {
  min-height: calc(100vh - 100px);
  display: flex;
  flex-direction: column;
}

.events-content {
  display: flex;
  flex: 1;
  gap: 1rem;
  align-items: flex-start;
  flex-wrap: nowrap; /* 保持两列布局 */
}

.search-bar-container {
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: flex-start;
  padding: 3rem 0 1rem 2.5rem;
  border-radius: 8px;
  width: 260px;
  flex: 0 0 260px;
  flex-shrink: 0;
}

.search-bar {
  color: #828282;
  width: 100%;
  padding: 0.8rem;
  border: 1px solid #000000;
  border-radius: 10px;
  box-shadow: 0 rgba(0, 0, 0, 0.1) 2px 4px;
  font-size: 1rem;
}

.search-btn {
  background-color: #6c63ff;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 0.8rem 1.2rem;
  margin-top: 0.5rem;
  cursor: pointer;
  transition: background-color 0.3s;
}

.search-btn:hover {
  background-color: #5753d6;
}

.events-page-with-search .el-main {
  margin: 0;
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0;
}

.event-list-container {
  flex: 1;
  display: flex;
  flex-direction: column;
}

/* 排序选择器样式 */
.sort-selector {
  display: flex;
  align-items: center;
  margin-top: 1rem;
  gap: 0.5rem;
  flex-wrap: nowrap; /* 强制不换行 */
}

.sort-label {
  font-size: 0.9rem;
  color: #666;
  font-weight: 500;
  white-space: nowrap; /* 标签文字不换行 */
}

.sort-dropdown {
  padding: 0.5rem 0.8rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: white;
  font-size: 0.9rem;
  color: #333;
  cursor: pointer;
  outline: none;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  min-width: 150px; /* 确保下拉框有最小宽度 */
}

.sort-dropdown:hover {
  border-color: #6c63ff;
  box-shadow: 0 2px 8px rgba(108, 99, 255, 0.15);
}

.sort-dropdown:focus {
  border-color: #6c63ff;
  box-shadow: 0 0 0 3px rgba(108, 99, 255, 0.1);
}

@media (max-width: 768px) {
  .events-page-with-search { 
    padding: 1rem; 
  }
}

@media (max-width: 576px) {
  
}
</style>
