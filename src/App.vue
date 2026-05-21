<template>
  <div id="app">
    <header class="header">
      <div class="container">
        <div class="logo">
          <h1>不良资产行业导航</h1>
          <p class="subtitle">专业不良资产信息平台</p>
        </div>
        <div class="search-box">
          <input 
            v-model="searchQuery" 
            type="text" 
            placeholder="搜索导航..." 
          />
        </div>
      </div>
    </header>

    <main class="main">
      <div class="container">
        <div class="stats">
          <div class="stat-item">
            <span class="stat-number">{{ totalLinks }}</span>
            <span class="stat-label">导航链接</span>
          </div>
          <div class="stat-item">
            <span class="stat-number">{{ categories.length }}</span>
            <span class="stat-label">分类数量</span>
          </div>
        </div>

        <div class="tag-filter" v-if="allTags.length > 0">
          <span class="tag-filter-label">标签筛选：</span>
          <div class="tag-filter-list">
            <button 
              class="tag-filter-btn"
              :class="{ active: activeTag === '' }"
              @click="activeTag = ''"
            >
              全部
            </button>
            <button 
              v-for="tag in allTags" 
              :key="tag"
              class="tag-filter-btn"
              :class="{ active: activeTag === tag }"
              @click="activeTag = tag"
            >
              {{ tag }}
            </button>
          </div>
        </div>

        <div class="categories">
          <CategorySection 
            v-for="category in filteredCategories" 
            :key="category.id"
            :category="category"
            :active-tag="activeTag"
            @tag-click="handleTagClick"
          />
        </div>

        <div v-if="filteredCategories.length === 0" class="no-results">
          <p>未找到相关导航，请尝试其他关键词</p>
        </div>
      </div>
    </main>

    <transition name="fade">
      <button 
        v-if="showBackTop" 
        class="back-top" 
        @click="scrollToTop"
        title="回到顶部"
      >
        ↑
      </button>
    </transition>

    <footer class="footer">
      <div class="container">
        <p>&copy; 2024 不良资产行业导航 | 专注不良资产领域信息服务</p>
      </div>
    </footer>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import CategorySection from './components/CategorySection.vue'
import { navigationData } from './data/navigation.js'

export default {
  name: 'App',
  components: {
    CategorySection
  },
  setup() {
    const searchQuery = ref('')
    const activeTag = ref('')
    const showBackTop = ref(false)
    const categories = ref(navigationData)

    const allTags = computed(() => {
      const tagSet = new Set()
      categories.value.forEach(category => {
        category.links.forEach(link => {
          link.tags.forEach(tag => tagSet.add(tag))
        })
      })
      return Array.from(tagSet).sort()
    })

    const filteredCategories = computed(() => {
      let result = categories.value

      if (activeTag.value) {
        result = result.map(category => {
          const filteredLinks = category.links.filter(link => 
            link.tags.includes(activeTag.value)
          )
          if (filteredLinks.length > 0) {
            return { ...category, links: filteredLinks }
          }
          return null
        }).filter(Boolean)
      }

      if (searchQuery.value.trim()) {
        const query = searchQuery.value.toLowerCase()
        result = result.map(category => {
          const filteredLinks = category.links.filter(link => 
            link.name.toLowerCase().includes(query) ||
            link.description.toLowerCase().includes(query) ||
            link.url.toLowerCase().includes(query)
          )
          if (filteredLinks.length > 0) {
            return { ...category, links: filteredLinks }
          }
          return null
        }).filter(Boolean)
      }

      return result
    })

    const totalLinks = computed(() => {
      return categories.value.reduce((total, category) => {
        return total + category.links.length
      }, 0)
    })

    const handleTagClick = (tag) => {
      activeTag.value = activeTag.value === tag ? '' : tag
    }

    const scrollToTop = () => {
      window.scrollTo({ top: 0, behavior: 'smooth' })
    }

    const handleScroll = () => {
      showBackTop.value = window.scrollY > 300
    }

    onMounted(() => {
      window.addEventListener('scroll', handleScroll)
    })

    onUnmounted(() => {
      window.removeEventListener('scroll', handleScroll)
    })

    return {
      searchQuery,
      activeTag,
      showBackTop,
      categories,
      filteredCategories,
      totalLinks,
      allTags,
      handleTagClick,
      scrollToTop
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen',
    'Ubuntu', 'Cantarell', 'Fira Sans', 'Droid Sans', 'Helvetica Neue',
    'Microsoft YaHei', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.header {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  padding: 30px 0;
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
  position: sticky;
  top: 0;
  z-index: 100;
}

.header .container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 20px;
}

.logo h1 {
  font-size: 28px;
  color: #2c3e50;
  font-weight: 700;
  margin-bottom: 5px;
}

.logo .subtitle {
  color: #7f8c8d;
  font-size: 14px;
}

.search-box input {
  width: 300px;
  padding: 12px 20px;
  border: 2px solid #e0e0e0;
  border-radius: 25px;
  font-size: 14px;
  transition: all 0.3s ease;
  outline: none;
}

.search-box input:focus {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.main {
  flex: 1;
  padding: 40px 0;
}

.stats {
  display: flex;
  justify-content: center;
  gap: 40px;
  margin-bottom: 30px;
}

.stat-item {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  padding: 20px 40px;
  border-radius: 15px;
  text-align: center;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.stat-number {
  display: block;
  font-size: 32px;
  font-weight: 700;
  color: #667eea;
  margin-bottom: 5px;
}

.stat-label {
  color: #7f8c8d;
  font-size: 14px;
}

.tag-filter {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 30px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.tag-filter-label {
  font-size: 14px;
  color: #2c3e50;
  font-weight: 600;
  margin-right: 10px;
}

.tag-filter-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 10px;
}

.tag-filter-btn {
  padding: 6px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 20px;
  background: #fff;
  color: #555;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.tag-filter-btn:hover {
  border-color: #667eea;
  color: #667eea;
}

.tag-filter-btn.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  border-color: transparent;
}

.categories {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.no-results {
  text-align: center;
  padding: 60px 20px;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 15px;
  color: #7f8c8d;
  font-size: 16px;
}

.back-top {
  position: fixed;
  bottom: 40px;
  right: 40px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  font-size: 22px;
  border: none;
  cursor: pointer;
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
  transition: all 0.3s ease;
  z-index: 99;
  display: flex;
  align-items: center;
  justify-content: center;
}

.back-top:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.5);
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.footer {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  padding: 20px 0;
  text-align: center;
  color: #7f8c8d;
  font-size: 14px;
}

@media (max-width: 768px) {
  .header .container {
    flex-direction: column;
    text-align: center;
  }

  .search-box input {
    width: 100%;
    max-width: 400px;
  }

  .stats {
    flex-direction: column;
    gap: 20px;
  }

  .stat-item {
    padding: 15px 30px;
  }

  .logo h1 {
    font-size: 24px;
  }

  .back-top {
    bottom: 20px;
    right: 20px;
    width: 44px;
    height: 44px;
    font-size: 18px;
  }
}
</style>
