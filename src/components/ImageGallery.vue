<template>
  <div class="gallery-container">
    <!-- 顶部控制栏 -->
    <div class="controls-wrapper" :class="{ 'scrolled': isScrolled }">
      <header>
        <h3>
          <span class="title-text">我的图库</span>
          <div class="title-underline"></div>
        </h3>
      </header>
      <div class="controls">
        <div class="search-box">
          <i class="fas fa-search search-icon"></i>
          <input 
            v-model="searchAuthor"
            type="text"
            placeholder="搜索作者..."
            @keyup.enter="handleSearch"
          >
          <button 
            class="search-btn" 
            @click="handleSearch"
            :disabled="isSearching"
          >
            <span>搜索</span>
          </button>
          
          <button 
                :class="['filter-btn', { active: r18Type === 1 }]"
                @click="toggleR18"
            >
                <i class="fas fa-heart"></i>
                <span>R18</span>
            </button>
        </div>
        

        <div class="view-toggle">
          <button 
            :class="['toggle-btn', { active: layout === 'masonry' }]"
            @click="layout = 'masonry'"
          >
            <i class="fas fa-th-large"></i>
            瀑布流
          </button>
          <button 
            :class="['toggle-btn', { active: layout === 'grid' }]"
            @click="layout = 'grid'"
          >
            <i class="fas fa-grip-horizontal"></i>
            单列
          </button>
        </div>
      </div>
    </div>

    <!-- 加载动画 -->
    <div v-if="loading" class="loading-container">
      <div class="loading-spinner"></div>
    </div>

    <!-- 错误提示 -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>

    <!-- 图片展示区 -->
    <div :class="['gallery', layout]">
      <div v-if="layout === 'masonry'" class="masonry-column" v-for="column in columns" :key="column">
        <TransitionGroup 
          name="fade"
          tag="div"
          class="column-content"
        >
          <div
            v-for="image in getColumnImages(column)"
            :key="image.pid"
            class="image-card"
            @click="openLightbox(image)"
          >
            <div 
              class="image-wrapper"
              :style="getImageStyle(image)"
            >
              <!-- 加载动画遮罩层 -->
              <div 
                class="image-loading-overlay"
                :class="{ 'loaded': loadedImageMap[image.pid] }"
              >
                <div class="loading-spinner"></div>
              </div>
              
              <img 
                :src="getThumbnailUrl(image.urlsList)"
                :alt="image.title"
                loading="eager"
                @load="onImageLoad(image.pid)"
              />
              <div class="image-overlay">
                <h3>{{ image.title }}</h3>
                <p>By: {{ image.author }}</p>
                <div class="tags">
                  <span v-for="tag in image.tagsList" :key="tag.tagName" class="tag">
                    {{ tag.tagName }}
                  </span>
                </div>
              </div>
            </div>
          </div>
          <!-- 预加载占位 -->
          <div 
            v-if="loadingMore && column === 1"
            key="loading-placeholder"
            class="image-card placeholder"
          >
            <div class="image-wrapper" style="paddingTop: 100%">
              <div class="image-loading-overlay">
                <div class="loading-spinner"></div>
              </div>
            </div>
          </div>
        </TransitionGroup>
      </div>
      <template v-else>
        <!-- 原有的grid布局 -->
        <div
          v-for="image in images"
          :key="image.pid"
          class="image-card"
          @click="openLightbox(image)"
        >
          <div class="image-wrapper" :style="getImageStyle(image)">
            <img 
              :src="getThumbnailUrl(image.urlsList)"
              :alt="image.title"
              loading="lazy"
              @load="onImageLoad"
            />
            <div class="image-overlay">
              <h3>{{ image.title }}</h3>
              <p>By: {{ image.author }}</p>
              <div class="tags">
                <span v-for="tag in image.tagsList" :key="tag.tagName" class="tag">
                  {{ tag.tagName }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </template>
    </div>

    <!-- 加载更多指示器 -->
    <div ref="loadMoreTrigger" class="load-more">
      <div v-if="loadingMore" class="loading-spinner"></div>
    </div>

    <!-- 图片预览弹窗 -->
    <div v-if="selectedImage" class="lightbox" @click="closeLightbox">
      <div class="lightbox-content" @click.stop>
        <div class="lightbox-scroll">
          <img 
            :src="getThumbnailUrl(selectedImage.urlsList, false)" 
            :alt="selectedImage.title"
            @load="onLightboxImageLoad"
          >
          <div class="lightbox-info">
            <h2>{{ selectedImage.title }}</h2>
            <p class="author">{{ selectedImage.author }}</p>
            <div class="tags">
              <span v-for="tag in selectedImage.tagsList" :key="tag.tagName" class="tag">
                {{ tag.tagName }}
              </span>
            </div>
          </div>
        </div>
        <div v-if="lightboxLoading" class="lightbox-loading">
          <div class="loading-spinner"></div>
        </div>
        <button class="close-btn" @click="closeLightbox">&times;</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'

// 新增的状态
const page = ref(1)
const loadingMore = ref(false)
const hasMore = ref(true)
const loadMoreTrigger = ref(null)
const observer = ref(null)
const loadedImages = ref(0)
const renderedCount = ref(0)
const renderInterval = ref(null)

// 修改现有的状态
const images = ref([])
const loading = ref(false)
const error = ref(null)
const layout = ref('grid')
const selectedImage = ref(null)

// 修改列数控制
const columns = ref(4)
const updateColumns = () => {
  if (window.innerWidth <= 440) {
    columns.value = 2  // 改为最少2列
  } else if (window.innerWidth <= 1024) {
    columns.value = 2
  } else if (window.innerWidth <= 1400) {
    columns.value = 3
  } else {
    columns.value = 4
  }
}

// 计算每列应显示的图片
const getColumnImages = (columnIndex) => {
  return visibleImages.value.filter((_, index) => index % columns.value === columnIndex - 1)
}

// 方法
const fetchImages = async (isLoadMore = false) => {
  if (!isLoadMore) {
    loading.value = true
    page.value = 1
    loadedImages.value = 0
    renderedCount.value = 0
    loadedImageMap.value = {}
  } else {
    loadingMore.value = true
  }

  try {
    const response = await fetch('https://api.mossia.top/duckMo', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ 
        num: 10,
        author: searchAuthor.value || undefined,
        r18Type: r18Type.value,
        sizeList: ['original', 'regular', 'small']
      })
    })

    // 检查状态码
    if (response.status === 429) {
      error.value = '请求太过频繁，请一个小时后再试'
      return
    }

    const data = await response.json()
    if (data.success) {
      if (isLoadMore) {
        images.value = [...images.value, ...data.data]
      } else {
        images.value = data.data
      }
      hasMore.value = data.data.length === 10
      startRendering()
    } else {
      error.value = data.message || '获取图片失败'
    }
  } catch (err) {
    error.value = err.message
  } finally {
    if (isLoadMore) {
      loadingMore.value = false
    } else {
      loading.value = false
    }
    isSearching.value = false
  }
}

const onImageLoad = (pid) => {
  loadedImageMap.value[pid] = true
  loadedImages.value++
  if (!renderInterval.value) {
    startRendering()
  }
}

const lightboxLoading = ref(false)

const onLightboxImageLoad = () => {
  lightboxLoading.value = false
}

const openLightbox = (image) => {
  selectedImage.value = image
  lightboxLoading.value = true
  document.body.style.overflow = 'hidden'
}

const closeLightbox = () => {
  selectedImage.value = null
  document.body.style.overflow = 'auto'
}

// 获取缩略图链接
const getThumbnailUrl = (itemImages, isThumbnail=true) => {
  // 如果有regular尺寸,优先使用regular作为缩略图
  const regularUrl = itemImages.find(url => url.urlSize === 'small')?.url
  if (isThumbnail && regularUrl) return regularUrl
  
  // 否则使用原图
  const originalUrl = itemImages.find(url => url.urlSize === 'original')?.url
  return originalUrl
}

// 新增加载更多方法
const loadMore = async () => {
  if (loadingMore.value || !hasMore.value) return
  
  loadingMore.value = true
  page.value++
  
  try {
    const response = await fetch('https://api.mossia.top/duckMo', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ 
        num: 5,
        author: searchAuthor.value || undefined,
        r18Type: r18Type.value,
        sizeList: ['original', 'regular', 'small']
      })
    })

    // 检查状态码
    if (response.status === 429) {
      error.value = '请求太过频繁，请一个小时后再试'
      return
    }
    
    const data = await response.json()
    if (data.success) {
      await new Promise(resolve => setTimeout(resolve, 300))
      images.value = [...images.value, ...data.data]
      hasMore.value = data.data.length === 5
      startRendering()
    }
  } catch (err) {
    error.value = err.message
  } finally {
    loadingMore.value = false
  }
}

// 设置 Intersection Observer
const setupIntersectionObserver = () => {
  observer.value = new IntersectionObserver(
    (entries) => {
      if (entries[0].isIntersecting) {
        loadMore()
      }
    },
    {
      // 当目标元素的 40% 进入视口时就触发
      threshold: 0.4,
      // 添加 rootMargin 使其提前触发
      rootMargin: '200px 0px'
    }
  )

  if (loadMoreTrigger.value) {
    observer.value.observe(loadMoreTrigger.value)
  }
}

// 生命周期钩子
onMounted(() => {
  fetchImages()
  setupIntersectionObserver()
  updateColumns()
  window.addEventListener('resize', updateColumns)
})

onUnmounted(() => {
  if (observer.value) {
    observer.value.disconnect()
  }
  window.removeEventListener('resize', updateColumns)
  if (renderInterval.value) {
    clearInterval(renderInterval.value)
  }
})

// 开始渲染图片
const startRendering = () => {
  renderInterval.value = setInterval(() => {
    if (renderedCount.value < images.value.length) {
      renderedCount.value++
    } else {
      clearInterval(renderInterval.value)
      renderInterval.value = null
    }
  }, 300)
}

// 获取当前应该显示的图片
const visibleImages = computed(() => {
  return images.value.slice(0, renderedCount.value)
})

// 添加已加载图片的映射
const loadedImageMap = ref({})

// 添加计算图片样式的方法
const getImageStyle = (image) => {
  const aspectRatio = image.height / image.width
  const paddingTop = `${aspectRatio * 100}%`
  return {
    paddingTop,
    background: '#f5f5f5'
  }
}

// 新增搜索相关的状态
const searchAuthor = ref('')
const isSearching = ref(false)

// 处理搜索
const handleSearch = () => {
  if (isSearching.value) return
  isSearching.value = true
  fetchImages()
}

// 添加 R18 状态控制
const r18Type = ref(0)

// 切换 R18 状态
const toggleR18 = () => {
  r18Type.value = r18Type.value === 0 ? 1 : 0
  fetchImages() // 重新获取图片
}
</script>
<style scoped>
.gallery-container {
  padding: 20px;
  max-width: 1400px;
  margin: 0 auto;
  min-height: 100vh;
  background: var(--bg-primary);
}

/* 控制栏容器 */
.controls-wrapper {
  position: sticky;
  top: 0;
  z-index: 100;
  background: var(--bg-primary);
  padding: 5px 20px;
  margin: -20px -20px 20px -20px;
  border-bottom: 1px solid var(--border-color);
  box-shadow: 0 2px 10px var(--shadow-color);
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
}

header {
    /* background: var(--bg-primary); */
    padding: 0.5rem;
    text-align: center;
    /* box-shadow: 0 2px 4px rgba(0,0,0,0.1); */
  }

/* 控制栏内容 */
.controls {
  max-width: 1400px;
  margin: 5px auto;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 20px;
}

/* 搜索框样式 */
.search-box {
  flex: 1;
  max-width: 500px;
  display: flex;
  align-items: center;
  gap: 10px;
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 8px 16px;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px var(--shadow-color);
}

.search-box:focus-within {
  /* border-color: var(--accent-color); */
  border-color: #4a90e2;
  box-shadow: 0 4px 12px rgba(74, 144, 226, 0.15);
  transform: translateY(-1px);
}

.search-icon {
  color: var(--text-secondary);
  font-size: 16px;
}

.search-box input {
  flex: 1;
  border: none;
  outline: none;
  padding: 8px;
  font-size: 15px;
  color: var(--text-primary);
  background: transparent;
}

.search-box input::placeholder {
  color: var(--text-secondary);
}

.search-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  background: var(--accent-color);
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.search-btn:not(:disabled):hover {
  background: var(--accent-hover);
  transform: translateY(-1px);
}

.search-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}


.search-btn:hover {
  color: #357abd;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .controls {
    flex-direction: column;
    gap: 15px;
  }
  
  .search-box {
    width: 100%;
    max-width: none;
  }
} 


/* 筛选按钮样式 */
.filter-btn {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  border: none;
  background: var(--bg-tertiary);
  color: var(--text-secondary);
  cursor: pointer;
  gap: 8px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.filter-btn i {
  font-size: 14px;
}

.filter-btn.active {
  background: var(--accent-color);
  color: white;
  box-shadow: 0 2px 8px var(--shadow-color);
}

.filter-btn:not(.active):hover {
  background: var(--bg-secondary);
  color: var(--accent-color);
  transform: translateY(-1px);
}


/* 布局切换按钮组 */
.view-toggle {
  display: flex;
  gap: 8px;
  background: var(--bg-tertiary);
  padding: 6px;
  border-radius: 10px;
}

.toggle-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  border: none;
  background: transparent;
  color: var(--text-secondary);
  font-size: 14px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.toggle-btn i {
  font-size: 16px;
}

.toggle-btn.active {
  background: var(--bg-secondary);
  color: var(--accent-color);
  box-shadow: 0 2px 8px var(--shadow-color);
}

.toggle-btn:not(.active):hover {
  background: var(--bg-secondary);
  opacity: 0.7;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .controls {
    flex-direction: column;
    gap: 15px;
  }
  
  .search-box {
    max-width: none;
  }
  
  .view-toggle {
    width: 100%;
  }
  
  .toggle-btn {
    flex: 1;
    justify-content: center;
  }
}

/* 图片网格布局 */
.gallery.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  width: 100%;
}
.gallery.grid image-wrapper {
    height: auto;
}

/* 瀑布流布局 */
.gallery.masonry {
  display: flex;
  gap: 20px;
  width: 100%;
  justify-content: center;
}

.masonry-column {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
  min-width: calc(50% - 10px);
  max-width: calc(50% - 10px);
}

.column-content {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.image-card {
  width: 100%;
  break-inside: avoid;
  border-radius: 8px;
  overflow: hidden;
  background: var(--bg-secondary);
  box-shadow: 0 2px 8px var(--shadow-color);
  transition: all 0.3s ease;
}

.image-wrapper {
  position: relative;
  width: 100%;
  background: var(--bg-tertiary);
}

.image-wrapper img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.image-wrapper img[src] {
  opacity: 1;
}

.image-loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: var(--overlay-bg);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1;
  transition: opacity 0.3s ease;
}

.image-loading-overlay.loaded {
  opacity: 0;
  pointer-events: none;
}

/* 响应式布局调整 */
@media (max-width: 1400px) {
  .masonry-column:nth-child(4) {
    display: none;
  }
}

@media (max-width: 1024px) {
  .masonry-column:nth-child(3) {
    display: none;
  }
}

@media (max-width: 440px) {
  .gallery-container {
    padding: 10px;
  }
  
  .masonry-column {
    gap: 10px;
  }
  
  .image-card {
    margin-bottom: 10px;
  }
}

/* 优化动画效果 */
.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(20px);
}

/* 悬停效果优化 */
.image-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.image-card:hover .image-overlay {
  opacity: 1;
  transform: translateY(0);
}

/* 图片卡片样式 */
.image-card {
  position: relative;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s, box-shadow 0.3s;
  cursor: pointer;
}

.image-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.2);
}

.image-wrapper {
  position: relative;
  overflow: hidden;
  background: #f5f5f5; /* 添加背景色 */
}

.image-wrapper img {
  width: 100%;
  height: auto;
  display: block;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.image-wrapper img[src] {
  opacity: 1;
}

.image-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.8), transparent);
  padding: 20px;
  color: white;
  opacity: 0;
  transform: translateY(10px);
  transition: all 0.3s ease;
}

.image-card:hover .image-overlay {
  opacity: 1;
  transform: translateY(0);
}

/* 标签样式 */
.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 10px;
}

.tag {
  background: rgba(255, 255, 255, 0.2);
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 0.8em;
}

/* 加载动画 */
.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 200px;
}

.loading-spinner {
  width: 50px;
  height: 50px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #4a90e2;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* 灯箱样式优化 */
.lightbox {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  animation: fadeIn 0.3s;
}

.lightbox-content {
  position: relative;
  width: 90%;
  height: 90vh;
  max-width: 1200px;
  background: var(--bg-secondary);
  border-radius: 8px;
  overflow: hidden;
  animation: scaleIn 0.3s;
}

.lightbox-scroll {
  height: 100%;
  overflow-y: auto;
  overflow-x: hidden;
  scrollbar-width: thin;
  scrollbar-color: #888 #f1f1f1;
}

/* 自定义滚动条样式 */
.lightbox-scroll::-webkit-scrollbar {
  width: 6px;
}

.lightbox-scroll::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.lightbox-scroll::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 3px;
}

.lightbox-scroll::-webkit-scrollbar-thumb:hover {
  background: #555;
}

.lightbox-content img {
  width: 100%;
  height: auto;
  display: block;
}

.lightbox-info {
  padding: 20px;
  background: var(--bg-secondary);
}

.lightbox-info h2 {
  margin: 0 0 10px 0;
  font-size: 24px;
  color: var(--text-primary);
}

.lightbox-info .author {
  margin: 0 0 15px 0;
  color: var(--text-secondary);
  font-size: 16px;
}

.lightbox-loading {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
}

.close-btn {
  position: absolute;
  top: 15px;
  right: 15px;
  width: 30px;
  height: 30px;
  border: none;
  background: rgba(0, 0, 0, 0.5);
  color: white;
  border-radius: 50%;
  font-size: 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.3s;
  z-index: 1001;
}

.close-btn:hover {
  background: rgba(0, 0, 0, 0.7);
}

/* 灯箱动画 */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes scaleIn {
  from { 
    transform: scale(0.9);
    opacity: 0;
  }
  to { 
    transform: scale(1);
    opacity: 1;
  }
}

/* 标签样式优化 */
.lightbox-info .tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.lightbox-info .tag {
  background: var(--bg-tertiary);
  color: var(--text-primary);
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 14px;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .lightbox-content {
    width: 100%;
    height: 100%;
    border-radius: 0;
  }

  .lightbox-info h2 {
    font-size: 20px;
  }

  .lightbox-info .author {
    font-size: 14px;
  }

  .lightbox-info .tag {
    font-size: 12px;
  }
}

/* 过渡动画 */
.grid-enter-active,
.grid-leave-active,
.masonry-enter-active,
.masonry-leave-active {
  transition: all 0.5s;
}

.grid-enter-from,
.grid-leave-to,
.masonry-enter-from,
.masonry-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.grid-move,
.masonry-move {
  transition: transform 0.5s;
}

/* 添加新的样式 */
.load-more {
  padding: 20px;
  text-align: center;
  margin-top: 20px;
}

.loading-spinner {
  width: 30px;
  height: 30px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #4a90e2;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* 添加渐入动画 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.column-content {
  gap: 20px;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

/* 加载动画遮罩层样式 */
.image-loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.9);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1;
  transition: opacity 0.3s ease;
}

.image-loading-overlay.loaded {
  opacity: 0;
  pointer-events: none;
}

/* 加载动画样式 */
.loading-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid #4a90e2;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

/* 标签样式优化 */
.tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 8px;
}

.tag {
  background: rgba(255, 255, 255, 0.2);
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  backdrop-filter: blur(4px);
}

/* 占位符样式 */
.image-card.placeholder {
  background: transparent;
  box-shadow: none;
}

.placeholder .image-wrapper {
  background: #f5f5f5;
  border-radius: 8px;
}

/* 优化过渡动画 */
.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.fade-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

.fade-move {
  transition: transform 0.3s ease;
}

/* 错误提示样式优化 */
.error-message {
  text-align: center;
  padding: 20px;
  margin: 20px 0;
  background: var(--bg-secondary);
  border-radius: 8px;
  color: #dc3545;
  font-size: 16px;
  box-shadow: 0 2px 8px var(--shadow-color);
}

</style>

