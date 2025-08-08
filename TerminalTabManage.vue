<template>
  <div class="tabs">
    <!-- 标签预览 -->
    <div v-if="previewTab" class="tab-preview" :style="previewStyle">
      <div class="preview-header">{{ previewTab.label }} - 备注</div>
      <div class="preview-content">
        {{ previewTab.remark || '暂无备注' }}
      </div>
    </div>
    <!-- 标签栏 -->
    <div class="tab-header" ref="tabHeader">
      <!-- 标签列表 -->
      <div
          v-for="(tab, index) in tabs"
          :key="tab.name"
          class="tab-item"
          :class="{ 
            active: activeTab === tab.name, 
            'history-tab': tab.name === 'connect',
            'drag-over': dragOverIndex === index && draggedIndex !== index
          }"
          :style="{ lineHeight: '30px'}"
          @click="activeTab = tab.name"
          @contextmenu.prevent="openContextMenu($event, tab.name)"
          :draggable="tab.name !== 'connect'"
          @dragstart="tab.name !== 'connect' && dragStart(index, $event)"
          @dragover.prevent="handleDragOver(index, $event)"
          @dragenter.prevent="handleDragEnter(index, $event)"
          @dragleave="handleDragLeave(index, $event)"
          @drop="drop(index, $event)"
          @dragend="handleDragEnd"
          @mouseenter="showPreview(tab, index, $event)"
          @mouseleave="hidePreview()"
      >
        <span class="tab-label">{{ tab.label}}</span>
        <span class="tab-close" @click.stop="closeTab(tab.name)">×</span>
      </div>

      <!-- 固定的添加按钮 -->
      <div
          class="tab-add"
          :style="{ lineHeight: '30px', width: tabWidth + 'px' }"
          @click="addTab"
      >
        ＋
      </div>
    </div>

    <!-- 内容区域 -->
    <div class="tab-content" :style="{ top: headerHeight + 'px' }">
      <div
          v-for="tab in tabs"
          :key="tab.name"
          v-show="activeTab === tab.name"
          class="tab-pane"
      >
        <component v-if="tab.component" :is="tab.component" />
        <template v-else>{{ tab.content }}</template>
      </div>
    </div>

    <!-- 右键菜单 -->
    <ul
        v-if="contextMenu.visible"
        class="context-menu"
        :style="{ top: contextMenu.y + 'px', left: contextMenu.x + 'px' }"
    >
      <li @click="closeTab(contextMenu.tab)">关闭当前</li>
      <li @click="closeOthers(contextMenu.tab)">关闭其他</li>
      <li @click="closeAll">关闭全部</li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, watch } from 'vue'
import ConnectManage from './ConnectManage.vue'

// 拖拽状态
const draggedIndex = ref(null)
const dragOverIndex = ref(null)

// 预览状态
const previewTab = ref(null)
const previewStyle = ref({
  top: '0px',
  left: '0px'
})
const previewTimer = ref(null)

const tabs = ref([
  { name: 'connect', label: '历史连接', component: ConnectManage, remark: '这里显示历史连接的备注信息，可以记录重要的连接信息和使用说明。' },
  { name: 'tab2', label: '标签 2', content: '这是标签 2 的内容', remark: '这是标签2的备注信息，可以记录一些重要的笔记和提醒。' }
])

const activeTab = ref('connect')

// 标签页配置
const tabHeader = ref(null)
const headerHeight = ref(30)
const tabWidth = 120

// 监听标签变化，重新计算标签栏高度
watch(tabs, async () => {
  await nextTick()
  updateHeaderHeight()
}, { deep: true })

// 计算标签栏的实际高度
function updateHeaderHeight() {
  if (tabHeader.value) {
    headerHeight.value = tabHeader.value.offsetHeight
  }
}

// 右键菜单状态
const contextMenu = ref({
  visible: false,
  x: 0,
  y: 0,
  tab: null
})

// 打开右键菜单
function openContextMenu(e, tabName) {
  contextMenu.value.visible = true
  contextMenu.value.x = e.clientX
  contextMenu.value.y = e.clientY
  contextMenu.value.tab = tabName
}

// 关闭标签
function closeTab(name) {
  const index = tabs.value.findIndex(tab => tab.name === name)
  if (index !== -1) {
    tabs.value.splice(index, 1)
    if (activeTab.value === name) {
      if (tabs.value[index]) {
        activeTab.value = tabs.value[index].name
      } else if (tabs.value[index - 1]) {
        activeTab.value = tabs.value[index - 1].name
      } else {
        activeTab.value = ''
      }
    }
  }
  
  // 如果所有标签页都关闭了，自动创建历史连接标签页
  if (tabs.value.length === 0) {
    createConnectTab()
  }
  
  hideContextMenu()
}

// 创建历史连接标签页
function createConnectTab() {
  tabs.value.push({
    name: 'connect',
    label: '历史连接',
    component: ConnectManage,
    remark: '这里显示历史连接的备注信息，可以记录重要的连接信息和使用说明。'
  })
  activeTab.value = 'connect'
}

// 关闭其他
function closeOthers(name) {
  tabs.value = tabs.value.filter(tab => tab.name === name)
  activeTab.value = name
  hideContextMenu()
}

// 关闭全部
function closeAll() {
  tabs.value = []
  // 关闭全部后自动创建历史连接标签页
  createConnectTab()
  hideContextMenu()
}

// 隐藏菜单
function hideContextMenu() {
  contextMenu.value.visible = false
}

// 添加新标签
let tabCounter = 3
function addTab() {
  const newName = `tab${tabCounter++}`
  const newTab = {
    name: newName,
    label: `标签 ${tabCounter - 1}`,
    content: `这是标签 ${tabCounter - 1} 的内容`,
    remark: `这是标签 ${tabCounter - 1} 的备注信息，可以在这里添加一些重要的笔记、提醒或者使用说明。`
  }
  
  // 检查历史连接标签页是否存在
  const connectIndex = tabs.value.findIndex(tab => tab.name === 'connect')
  
  if (connectIndex === -1) {
    // 如果不存在历史连接标签页，先创建它
    createConnectTab()
    // 然后在它后面添加新标签
    tabs.value.splice(1, 0, newTab)
  } else {
    // 如果存在历史连接标签页，在它后面添加新标签
    tabs.value.splice(connectIndex + 1, 0, newTab)
  }
  
  activeTab.value = newName
}

// 拖拽开始
function dragStart(index, event) {
  draggedIndex.value = index
  event.dataTransfer.effectAllowed = 'move'
  event.dataTransfer.setData('text/plain', index.toString())
  // 添加拖拽样式
  event.target.classList.add('dragging')
}

// 拖拽经过
function handleDragOver(index, event) {
  if (index === 0 && tabs.value[0].name === 'connect') {
    // 不允许拖拽到历史连接标签上
    return
  }
  
  if (draggedIndex.value !== null && draggedIndex.value !== index) {
    dragOverIndex.value = index
  }
}

// 拖拽进入
function handleDragEnter(index, event) {
  if (index === 0 && tabs.value[0].name === 'connect') {
    // 不允许拖拽到历史连接标签上
    return
  }
  
  if (draggedIndex.value !== null && draggedIndex.value !== index) {
    dragOverIndex.value = index
  }
}

// 拖拽离开
function handleDragLeave(index, event) {
  // 检查是否真的离开了元素（而不是进入了子元素）
  const rect = event.target.getBoundingClientRect()
  const x = event.clientX
  const y = event.clientY
  
  if (x < rect.left || x >= rect.right || y < rect.top || y >= rect.bottom) {
    if (dragOverIndex.value === index) {
      dragOverIndex.value = null
    }
  }
}

// 拖拽结束
function handleDragEnd() {
  draggedIndex.value = null
  dragOverIndex.value = null
  
  // 移除所有元素的拖拽样式
  const tabItems = document.querySelectorAll('.tab-item')
  tabItems.forEach(item => item.classList.remove('dragging'))
}

// 放置处理
function drop(targetIndex, event) {
  const sourceIndex = parseInt(event.dataTransfer.getData('text/plain'))
  
  // 检查是否是历史连接标签页
  const isSourceConnect = tabs.value[sourceIndex].name === 'connect'
  const isTargetConnect = tabs.value[targetIndex].name === 'connect'
  
  // 如果源标签是历史连接或目标位置是历史连接的位置（第一个），则不执行拖拽
  if (isSourceConnect || (targetIndex === 0 && !isTargetConnect)) {
    // 重置拖拽状态
    draggedIndex.value = null
    dragOverIndex.value = null
    return
  }
  
  if (sourceIndex !== targetIndex) {
    // 获取要移动的标签
    const movedTab = tabs.value[sourceIndex]
    
    // 从数组中移除该标签
    tabs.value.splice(sourceIndex, 1)
    
    // 在目标位置插入该标签，确保不会插入到历史连接标签页之前
    const actualTargetIndex = targetIndex === 0 ? 1 : targetIndex
    tabs.value.splice(actualTargetIndex, 0, movedTab)
  }
  
  // 重置拖拽状态
  draggedIndex.value = null
  dragOverIndex.value = null
}

// 显示预览
function showPreview(tab, index, event) {
  // 清除之前的定时器
  if (previewTimer.value) {
    clearTimeout(previewTimer.value)
  }
  
  // 设置延迟显示预览，避免鼠标快速经过时频繁显示
  previewTimer.value = setTimeout(() => {
    // 如果是当前激活的标签，不显示预览
    if (tab.name === activeTab.value) {
      return
    }
    
    // 计算预览位置
    const rect = event.target.getBoundingClientRect()
    previewStyle.value = {
      top: rect.bottom + 5 + 'px',
      left: rect.left + 'px'
    }
    
    // 设置预览内容
    previewTab.value = tab
  }, 500) // 500ms 延迟
}

// 隐藏预览
function hidePreview() {
  if (previewTimer.value) {
    clearTimeout(previewTimer.value)
  }
  
  // 设置延迟隐藏预览，避免鼠标移动到预览框时立即消失
  previewTimer.value = setTimeout(() => {
    previewTab.value = null
  }, 300) // 300ms 延迟
}

onMounted(() => {
  document.addEventListener('click', hideContextMenu)
  updateHeaderHeight()
})
</script>

<style scoped>
.tabs {
  position: relative;
  display: flex;
  flex-direction: column;
  border: none;
  height: 100%;
  width: 100%;
  background-color: rgba(36, 189, 120, 0.67);
  color: #eaeaea;
}

.tab-header {
  display: flex;
  flex-wrap: wrap;
  background: #252526;
  border-bottom: 1px solid #333;
  overflow: visible;
  position: relative;
  padding-right: 40px; /* 为固定位置的添加按钮留出空间 */
}

.tab-item {
  font-size: 12px;
  position: relative;
  flex: none;
  text-align: center;
  cursor: pointer;
  user-select: none;
  border-right: 1px solid #333;
  color: #ccc;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  transition: background-color 0.2s;
}

.tab-item.dragging {
  opacity: 0.7;
  background-color: #3c3c3c;
  z-index: 100;
}

.tab-item.drag-over {
  transform: translateX(10px);
  position: relative;
}

.tab-item {
  transition: transform 0.2s ease, background-color 0.2s;
}

.tab-item.history-tab {
  background-color: #2d4b65;
  border-bottom: 2px solid #00a2ff;
  cursor: default;
}

.tab-item.history-tab.active {
  background-color: #1e3a52;
}

.tab-item.active {
  background: #1e1e1e;
  font-weight: bold;
  border-bottom: 2px solid #0078d4;
  color: #fff;
}

.tab-add {
  position: absolute;
  right: 0;
  top: 0;
  width: 40px !important;
  text-align: center;
  background: #2d2d2d;
  cursor: pointer;
  user-select: none;
  border-left: 1px solid #333;
  color: #ccc;
  z-index: 10;
}

.tab-add:hover {
  background: #3c3c3c;
  color: #fff;
}

.tab-label {
  flex: none;
  padding: 0 20px 0 10px;
}

.tab-close {
  position: absolute;
  right: 5px;
  top: 0;
  color: #888;
  font-weight: bold;
  cursor: pointer;
}

.tab-close:hover {
  color: red;
}

.tab-content {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: auto;
  padding: 0;
  background-color: rgb(80, 91, 109);
}

.tab-pane {
  height: 100%;
  width: 100%;
  padding: 10px;
  box-sizing: border-box;
}

.context-menu {
  position: fixed;
  list-style: none;
  margin: 0;
  padding: 5px 0;
  background: #252526;
  border: 1px solid #333;
  box-shadow: 0 2px 8px rgba(0,0,0,0.5);
  z-index: 9999;
  color: #eaeaea;
}

.context-menu li {
  padding: 5px 20px;
  cursor: pointer;
}

.context-menu li:hover {
  background: #0078d4;
}

/* 标签预览样式 */
.tab-preview {
  position: fixed;
  width: 300px;
  max-height: 200px;
  background-color: #1e1e1e;
  border: 1px solid #444;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
  z-index: 1000;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.preview-header {
  padding: 8px 12px;
  background-color: #252526;
  border-bottom: 1px solid #444;
  font-weight: bold;
  color: #fff;
}

.preview-content {
  padding: 10px;
  overflow: auto;
  flex: 1;
  max-height: 160px;
  font-size: 12px;
}
</style>
