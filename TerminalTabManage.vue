<template>
  <div class="tabs">
    <!-- 拖拽预览线 -->
    <div v-if="dragOverIndex !== null" class="drag-preview-line" :style="dragPreviewLineStyle"></div>
    
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
          v-for="(tab, index) in tabs.filter(tab => !tab.hidden)"
          :key="tab.name"
          class="tab-item"
          :class="{ 
            active: activeTab === tab.name, 
            'history-tab': tab.name === 'connect',
            'drag-over': dragOverIndex === index && draggedIndex !== index
          }"
          :style="{ lineHeight: '25px'}"
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
          :style="{ lineHeight: '25px', width: tabWidth + 'px' }"
          @click="addTab"
      >
        ＋
      </div>
    </div>

    <!-- 内容区域 -->
    <div class="tab-content" :style="{ top: headerHeight + 'px', bottom: '0px', left: '0px', right: '0px' }">
      <div
          v-for="tab in tabs"
          :key="tab.name"
          v-show="activeTab === tab.name || tab.hidden"
          class="tab-pane"
          :style="{ visibility: tab.hidden ? 'hidden' : 'visible', position: tab.hidden ? 'absolute' : 'relative', zIndex: tab.hidden ? -1 : 'auto' }"
      >
        <!-- 恢复所有组件 -->
        <component 
          v-if="tab.component" 
          :is="tab.component" 
          :key="`${tab.name}-${tab.uniqueId || ''}`"
        />
        <template v-else>{{ tab.content }}</template>
      </div>
    </div>

    <!-- 右键菜单 -->
    <ul
        style="font-size: 12px; border-radius: 10px"
        v-if="contextMenu.visible"
        class="context-menu"
        :style="{ top: contextMenu.y + 'px', left: contextMenu.x + 'px'}"
    >
      <li @click="closeTab(contextMenu.tab)">关闭当前</li>
      <li @click="closeOthers(contextMenu.tab)">关闭其他</li>
      <li @click="closeAll">关闭全部</li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted, nextTick, watch, markRaw, defineAsyncComponent } from 'vue'
// 恢复快速连接组件的导入
import ConnectManage from './ConnectManage.vue'

// 使用异步组件加载终端分割器，避免重复实例化
const TerminalHSplitter = defineAsyncComponent(() => 
  import('./TerminalHSplitter.vue')
)

// 拖拽状态
const draggedIndex = ref(null)
const dragOverIndex = ref(null)
const dragPreviewLineStyle = ref({
  left: '0px',
  height: '0px',
  top: '0px'
})

// 预览状态
const previewTab = ref(null)
const previewStyle = ref({
  top: '0px',
  left: '0px'
})
const previewTimer = ref(null)

const tabs = ref([
  { 
    name: 'connect', 
    label: '快速连接', 
    component: markRaw(ConnectManage), // 恢复组件引用
    remark: '这里显示快速连接的备注信息，可以记录重要的连接信息和使用说明。' 
  },
  // 恢复预加载的隐藏终端标签
  { 
    name: 'preloaded', 
    label: '预加载终端', 
    component: markRaw(TerminalHSplitter), 
    remark: '预加载终端，确保终端在启动时就初始化好', 
    hidden: true,
    uniqueId: 'preloaded-terminal'
  }
])

const activeTab = ref('connect')

// 标签页配置
const tabHeader = ref(null)
const headerHeight = ref(25)
const tabWidth = 120

// 监听标签变化，重新计算标签栏高度
watch(tabs, async () => {
  await nextTick()
  updateHeaderHeight()
  
  // 标签变化时触发终端重新适配事件
  const delays = [50, 200, 500];
  delays.forEach(delay => {
    setTimeout(() => {
      window.dispatchEvent(new Event('terminal-resize'));
    }, delay);
  });
}, { deep: true })

// 监听活动标签变化
watch(activeTab, () => {
  // 标签切换时触发终端重新适配事件
  const delays = [50, 200, 500];
  delays.forEach(delay => {
    setTimeout(() => {
      window.dispatchEvent(new Event('terminal-resize'));
    }, delay);
  });
})

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
  // 不允许关闭预加载的隐藏终端标签
  if (name === 'preloaded') {
    return;
  }
  
  // 如果当前有预览窗口，先隐藏它
  hidePreview();
  
  // 在关闭标签前，获取所有可见标签
  const visibleTabs = tabs.value.filter(tab => !tab.hidden);
  
  // 找到当前关闭标签在可见标签中的索引
  const visibleIndex = visibleTabs.findIndex(tab => tab.name === name);
  
  // 保存当前是否是激活标签
  const isActiveTab = activeTab.value === name;
  
  // 关闭标签
  const index = tabs.value.findIndex(tab => tab.name === name);
  if (index !== -1) {
    tabs.value.splice(index, 1);
    
    // 如果关闭的是当前激活的标签，需要激活另一个标签
    if (isActiveTab) {
      // 获取更新后的可见标签列表
      const updatedVisibleTabs = tabs.value.filter(tab => !tab.hidden);
      
      if (updatedVisibleTabs.length > 0) {
        // 尝试激活前一个标签
        if (visibleIndex > 0 && visibleIndex <= updatedVisibleTabs.length) {
          // 激活前一个标签
          activeTab.value = updatedVisibleTabs[visibleIndex - 1].name;
        } 
        // 如果关闭的是第一个标签，激活新的第一个标签
        else if (visibleIndex === 0 && updatedVisibleTabs.length > 0) {
          activeTab.value = updatedVisibleTabs[0].name;
        }
        // 其他情况，激活第一个可见标签
        else {
          activeTab.value = updatedVisibleTabs[0].name;
        }
      } else {
        // 如果没有可见标签，设置为空
        activeTab.value = '';
      }
    }
  }
  
  // 检查是否还有可见的标签页（除了快速连接标签）
  const visibleNonConnectTabs = tabs.value.filter(tab => !tab.hidden && tab.name !== 'connect');
  
  // 如果没有可见的非快速连接标签，并且也没有快速连接标签，则创建快速连接标签
  const hasConnectTab = tabs.value.some(tab => tab.name === 'connect');
  if (visibleNonConnectTabs.length === 0 && !hasConnectTab) {
    createConnectTab();
  }
  
  hideContextMenu();
}

// 创建快速连接标签页
function createConnectTab() {
  // 始终将快速连接标签添加到数组的开头
  tabs.value.unshift({
    name: 'connect',
    label: '快速连接',
    component: markRaw(ConnectManage), // 恢复组件引用
    remark: '这里显示快速连接的备注信息，可以记录重要的连接信息和使用说明。'
  })
  activeTab.value = 'connect'
}

// 关闭其他
function closeOthers(name) {
  // 隐藏预览窗口
  hidePreview()
  
  // 保留当前标签和预加载的隐藏终端标签
  tabs.value = tabs.value.filter(tab => tab.name === name || tab.name === 'preloaded')
  activeTab.value = name
  hideContextMenu()
}

// 关闭全部
function closeAll() {
  // 隐藏预览窗口
  hidePreview()
  
  // 只保留预加载的隐藏终端标签
  tabs.value = tabs.value.filter(tab => tab.name === 'preloaded')
  
  // 关闭全部后自动创建快速连接标签页
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
  const uniqueId = `terminal-${Date.now()}`
  
  // 恢复使用TerminalHSplitter组件
  const newTab = {
    name: newName,
    label: `标签 ${tabCounter - 1}`,
    component: markRaw(TerminalHSplitter),
    remark: `这是标签 ${tabCounter - 1} 的备注信息，可以在这里添加一些重要的笔记、提醒或者使用说明。`,
    uniqueId: uniqueId // 添加唯一ID，确保每个终端实例都是独立的
  }
  
  // 找到预加载标签的索引
  const preloadedIndex = tabs.value.findIndex(tab => tab.name === 'preloaded')
  
  // 将新标签添加到预加载标签之前（如果存在），否则添加到末尾
  if (preloadedIndex !== -1) {
    tabs.value.splice(preloadedIndex, 0, newTab)
  } else {
    tabs.value.push(newTab)
  }
  
  // 激活新标签
  activeTab.value = newName
  
  // 添加标签后，使用多个延迟触发终端重新适配事件，确保DOM已完全更新
  const delays = [100, 300, 500, 1000];
  delays.forEach(delay => {
    setTimeout(() => {
      window.dispatchEvent(new Event('terminal-resize'));
    }, delay);
  });
  
  console.log(`添加了新标签: ${newName}, ID: ${uniqueId}`);
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
    updateDragPreviewLine(event.currentTarget)
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
    updateDragPreviewLine(event.currentTarget)
  }
}

// 更新拖拽预览线位置
function updateDragPreviewLine(element) {
  if (!element) return
  
  const rect = element.getBoundingClientRect()
  
  // 如果拖拽的索引小于目标索引，预览线显示在目标右侧
  if (draggedIndex.value < dragOverIndex.value) {
    dragPreviewLineStyle.value = {
      left: (rect.right) + 'px',
      height: rect.height + 'px',
      top: rect.top + 'px'
    }
  } else {
    // 否则显示在左侧
    dragPreviewLineStyle.value = {
      left: (rect.left) + 'px',
      height: rect.height + 'px',
      top: rect.top + 'px'
    }
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
  
  // 检查是否是快速连接标签页
  const isSourceConnect = tabs.value[sourceIndex].name === 'connect'
  const isTargetConnect = tabs.value[targetIndex].name === 'connect'
  
  // 如果源标签是快速连接或目标位置是快速连接的位置（第一个），则不执行拖拽
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
    
    // 在目标位置插入该标签，确保不会插入到快速连接标签页之前
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
    
    // 如果只有一个标签页，不显示预览
    if (tabs.value.length <= 1) {
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
  }, 1000) // 1000ms (1秒) 延迟
}

// 隐藏预览
function hidePreview() {
  if (previewTimer.value) {
    clearTimeout(previewTimer.value)
    previewTimer.value = null
  }
  
  // 立即隐藏预览
  previewTab.value = null
  
  // 重置预览样式
  previewStyle.value = {
    top: '0px',
    left: '0px'
  }
}

onMounted(() => {
  document.addEventListener('click', hideContextMenu)
  updateHeaderHeight()
})
</script>

<style scoped>
.tabs {
  position: absolute;
  display: flex;
  flex-direction: column;
  border: none;
  height: 100%;
  width: 100%;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: hidden;
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
  background: #0078d4;
  border-bottom: none;
  cursor: default;
  color: #fff;
}

.tab-item.history-tab.active {
  background: #0078d4;
  border-bottom: none;
  color: #fff;
}

.tab-item.active:not(.history-tab) {
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
  top: 25px; /* 与 headerHeight 保持一致 */
  overflow: hidden; /* 改为 hidden，防止出现滚动条 */
  padding: 0;
  background-color: rgb(80, 91, 109);
  scrollbar-width: thin; /* Firefox */
  scrollbar-color: #555 #2a2a2a; /* Firefox */
}

.tab-pane {
  height: 100%;
  width: 100%;
  padding: 0;
  box-sizing: border-box;
  overflow: hidden; /* 防止内容溢出 */
  position: absolute; /* 使用绝对定位填充整个容器 */
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
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

/* 拖拽预览线样式 */
.drag-preview-line {
  position: fixed;
  width: 2px;
  background-color: #0078d4;
  box-shadow: 0 0 5px rgba(0, 120, 212, 0.7);
  z-index: 1000;
  pointer-events: none;
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
  scrollbar-width: thin; /* Firefox */
  scrollbar-color: #555 #2a2a2a; /* Firefox */
}
</style>