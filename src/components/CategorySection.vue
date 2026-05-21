<template>
  <section class="category-section">
    <div class="category-header" @click="toggleCollapse">
      <div class="category-icon">{{ category.icon }}</div>
      <h2 class="category-title">{{ category.name }}</h2>
      <span class="category-count">{{ category.links.length }} 个链接</span>
      <span class="collapse-btn" :class="{ collapsed: isCollapsed }">▼</span>
    </div>
    
    <transition name="collapse">
      <div class="links-grid" v-show="!isCollapsed">
        <a 
          v-for="link in category.links" 
          :key="link.url"
          :href="link.url"
          target="_blank"
          rel="noopener noreferrer"
          class="link-card"
        >
          <div class="link-header">
            <div class="link-title-row">
              <img 
                :src="getFavicon(link.url)" 
                :alt="link.name"
                class="link-favicon"
                @error="handleFaviconError"
              />
              <h3 class="link-name">{{ link.name }}</h3>
            </div>
            <span class="link-arrow">→</span>
          </div>
          <p class="link-description">{{ link.description }}</p>
          <div class="link-tags">
            <span 
              v-for="tag in link.tags" 
              :key="tag" 
              class="link-tag"
              :class="{ active: activeTag === tag }"
              @click.prevent="onTagClick(tag)"
            >
              {{ tag }}
            </span>
          </div>
        </a>
      </div>
    </transition>
  </section>
</template>

<script>
import { ref } from 'vue'

export default {
  name: 'CategorySection',
  props: {
    category: {
      type: Object,
      required: true
    },
    activeTag: {
      type: String,
      default: ''
    }
  },
  emits: ['tag-click'],
  setup(props, { emit }) {
    const isCollapsed = ref(false)

    const toggleCollapse = () => {
      isCollapsed.value = !isCollapsed.value
    }

    const getFavicon = (url) => {
      try {
        const domain = new URL(url).hostname
        return `https://www.google.com/s2/favicons?domain=${domain}&sz=32`
      } catch {
        return ''
      }
    }

    const handleFaviconError = (e) => {
      e.target.style.display = 'none'
    }

    const onTagClick = (tag) => {
      emit('tag-click', tag)
    }

    return {
      isCollapsed,
      toggleCollapse,
      getFavicon,
      handleFaviconError,
      onTagClick
    }
  }
}
</script>

<style scoped>
.category-section {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.category-header {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 25px;
  padding-bottom: 20px;
  border-bottom: 2px solid #f0f0f0;
  cursor: pointer;
  user-select: none;
}

.category-header:hover .collapse-btn {
  color: #667eea;
}

.category-icon {
  font-size: 32px;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  flex-shrink: 0;
}

.category-title {
  font-size: 24px;
  color: #2c3e50;
  font-weight: 700;
  flex: 1;
}

.category-count {
  color: #7f8c8d;
  font-size: 14px;
  background: #f8f9fa;
  padding: 5px 15px;
  border-radius: 20px;
  flex-shrink: 0;
}

.collapse-btn {
  color: #999;
  font-size: 14px;
  transition: all 0.3s ease;
  flex-shrink: 0;
}

.collapse-btn.collapsed {
  transform: rotate(-90deg);
}

.links-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
  overflow: hidden;
}

.collapse-enter-active,
.collapse-leave-active {
  transition: all 0.3s ease;
  max-height: 2000px;
}

.collapse-enter-from,
.collapse-leave-to {
  max-height: 0;
  opacity: 0;
}

.link-card {
  background: #fff;
  border: 2px solid #f0f0f0;
  border-radius: 12px;
  padding: 20px;
  text-decoration: none;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.link-card:hover {
  border-color: #667eea;
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.15);
  transform: translateY(-3px);
}

.link-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.link-title-row {
  display: flex;
  align-items: center;
  gap: 10px;
  min-width: 0;
}

.link-favicon {
  width: 20px;
  height: 20px;
  border-radius: 4px;
  flex-shrink: 0;
  object-fit: contain;
}

.link-name {
  font-size: 16px;
  color: #2c3e50;
  font-weight: 600;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.link-arrow {
  color: #667eea;
  font-size: 18px;
  transition: transform 0.3s ease;
  flex-shrink: 0;
}

.link-card:hover .link-arrow {
  transform: translateX(5px);
}

.link-description {
  color: #7f8c8d;
  font-size: 13px;
  line-height: 1.5;
}

.link-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 5px;
}

.link-tag {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  font-size: 11px;
  padding: 3px 10px;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.link-tag:hover {
  opacity: 0.8;
  transform: scale(1.05);
}

.link-tag.active {
  background: #e74c3c;
  box-shadow: 0 2px 8px rgba(231, 76, 60, 0.3);
}

@media (max-width: 768px) {
  .category-section {
    padding: 20px;
  }

  .category-header {
    flex-wrap: wrap;
  }

  .category-title {
    font-size: 20px;
  }

  .links-grid {
    grid-template-columns: 1fr;
  }
}
</style>
