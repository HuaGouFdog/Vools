<template>
  <div class="terminal-vsplitter" ref="splitterContainer">
    <div class="panel top-panel" :style="{ height: topPanelHeight + 'px' }">
      <!-- 使用动态组件和唯一ID -->
      <component 
        :is="terminalMainComponent" 
        :key="`terminal-main-${uniqueId}`"
        :uniqueId="uniqueId"
      />
    </div>
    <div
      class="splitter-handle"
      @mousedown.stop="startDrag"
      :style="{ top: topPanelHeight + 'px' }"
    ></div>
    <div class="panel bottom-panel" :style="{ height: bottomPanelHeight + 'px' }">
      <!-- 使用动态组件和唯一ID -->
      <component 
        :is="terminalHSplitter2Component" 
        :key="`terminal-hsplitter2-${uniqueId}`"
        :uniqueId="uniqueId"
      >
        <template #right>
          <slot name="bottom"></slot>
        </template>
      </component>
    </div>
  </div>
</template>

<script>
import { defineAsyncComponent, markRaw } from 'vue';

// 使用异步组件和markRaw减少响应式开销
const TerminalMain = defineAsyncComponent(() => import('./TerminalMain.vue'));
const TerminalHSplitter2 = defineAsyncComponent(() => import('./TerminalHSplitter2.vue'));

// 存储每个实例的布局状态
const instanceStates = new Map();

export default {
  name: 'TerminalVSplitter',
  props: {
    // 允许父组件传入唯一标识符
    uniqueId: {
      type: String,
      default: () => `vsplit-${Date.now()}-${Math.random().toString(36).substring(2, 9)}`
    },
    // 初始分割比例，范围0-1，表示上面板所占比例
    initialRatio: {
      type: Number,
      default: 0.75 // 3:1的比例，上部分占3/4
    },
    // 最小面板高度（像素）
    minPanelHeight: {
      type: Number,
      default: 50
    }
  },
  data() {
    return {
      // 上面板高度
      topPanelHeight: 0,
      // 下面板高度
      bottomPanelHeight: 0,
      // 容器总高度
      containerHeight: 0,
      // 拖动状态
      isDragging: false,
      // 使用markRaw包装组件，避免不必要的响应式处理
      terminalMainComponent: markRaw(TerminalMain),
      terminalHSplitter2Component: markRaw(TerminalHSplitter2),
      // 本地事件处理函数引用
      localHandleResize: null,
      localStopDrag: null,
      localHandleDrag: null,
      localTerminalResize: null,
      // 初始化状态
      isInitialized: false,
      // 当前分割比例
      currentRatio: this.initialRatio,
      // 可见状态
      isVisible: true,
      // 观察器
      resizeObserver: null
    }
  },
  created() {
    // 创建本地事件处理函数，避免内存泄漏
    this.localHandleResize = this.handleResize.bind(this);
    this.localStopDrag = this.stopDrag.bind(this);
    this.localHandleDrag = this.handleDrag.bind(this);
    this.localTerminalResize = this.handleTerminalResize.bind(this);
    
    // 从存储中恢复状态（如果有）
    if (instanceStates.has(this.uniqueId)) {
      const savedState = instanceStates.get(this.uniqueId);
      this.currentRatio = savedState.ratio;
      console.log(`恢复VSplitter状态: ${this.uniqueId}, 比例: ${this.currentRatio}`);
    }
    
    console.log(`创建VSplitter实例: ${this.uniqueId}`);
  },
  mounted() {
    // 创建ResizeObserver监听容器大小变化
    this.resizeObserver = new ResizeObserver(entries => {
      if (entries.length > 0) {
        this.handleContainerResize();
      }
    });
    
    // 开始观察容器大小变化
    if (this.$refs.splitterContainer) {
      this.resizeObserver.observe(this.$refs.splitterContainer);
    }
    
    // 使用多个延迟初始化面板，确保在各种情况下都能正确初始化
    const delays = [50, 150, 300, 500, 1000];
    delays.forEach(delay => {
      setTimeout(() => {
        if (!this.isInitialized || delay >= 300) {
          this.initializePanels();
          this.isInitialized = true;
        }
      }, delay);
    });

    // 使用本地事件处理函数
    window.addEventListener('resize', this.localHandleResize);
    document.addEventListener('mouseup', this.localStopDrag);
    document.addEventListener('mousemove', this.localHandleDrag);
    window.addEventListener('terminal-resize', this.localTerminalResize);
    
    // 添加可见性变化监听
    document.addEventListener('visibilitychange', this.handleVisibilityChange);
    
    console.log(`VSplitter挂载完成: ${this.uniqueId}`);
  },
  beforeUnmount() {
    // 保存当前状态
    instanceStates.set(this.uniqueId, {
      ratio: this.currentRatio
    });
    
    // 停止观察容器大小变化
    if (this.resizeObserver) {
      this.resizeObserver.disconnect();
      this.resizeObserver = null;
    }
    
    // 移除所有事件监听器
    window.removeEventListener('resize', this.localHandleResize);
    document.removeEventListener('mouseup', this.localStopDrag);
    document.removeEventListener('mousemove', this.localHandleDrag);
    window.removeEventListener('terminal-resize', this.localTerminalResize);
    document.removeEventListener('visibilitychange', this.handleVisibilityChange);
    
    // 清除所有引用
    this.localHandleResize = null;
    this.localStopDrag = null;
    this.localHandleDrag = null;
    this.localTerminalResize = null;
    
    console.log(`VSplitter卸载: ${this.uniqueId}`);
  },
  methods: {
    // 处理可见性变化
    handleVisibilityChange() {
      const isVisible = document.visibilityState === 'visible';
      if (isVisible && !this.isVisible) {
        // 从不可见变为可见时，重新计算布局
        this.$nextTick(() => {
          this.handleContainerResize();
        });
      }
      this.isVisible = isVisible;
    },
    
    // 处理容器大小变化
    handleContainerResize() {
      if (!this.$refs.splitterContainer) return;
      
      const newHeight = this.$refs.splitterContainer.clientHeight;
      if (newHeight === 0) return;
      
      this.containerHeight = newHeight;
      
      // 使用保存的比例计算面板高度
      this.topPanelHeight = Math.max(
        this.minPanelHeight,
        Math.min(
          this.containerHeight - this.minPanelHeight,
          this.containerHeight * this.currentRatio
        )
      );
      
      this.bottomPanelHeight = this.containerHeight - this.topPanelHeight;
      
      console.log(`容器大小变化: ${this.uniqueId}, 高度: ${this.containerHeight}, 比例: ${this.currentRatio}`);
    },
    
    // 处理终端重新适配事件，减少事件触发频率
    handleTerminalResize() {
      // 仅在组件可见时处理
      if (!this.isVisible) return;
      
      // 使用防抖，减少事件触发频率
      if (this.terminalResizeTimeout) {
        clearTimeout(this.terminalResizeTimeout);
      }
      
      // 使用多个延迟，确保在各种情况下都能正确调整大小
      const delays = [50, 150, 300, 500];
      delays.forEach(delay => {
        setTimeout(() => {
          this.handleContainerResize();
        }, delay);
      });
    },
    
    // 初始化面板尺寸
    initializePanels() {
      // 确保容器元素存在
      if (!this.$refs.splitterContainer) {
        console.warn(`分割容器元素不存在，使用默认高度 (${this.uniqueId})`);
        this.containerHeight = 600; // 提供默认高度
      } else {
        this.containerHeight = this.$refs.splitterContainer.clientHeight || 600;
      }

      // 使用当前比例计算面板高度
      this.topPanelHeight = Math.max(
        this.minPanelHeight,
        Math.min(
          this.containerHeight - this.minPanelHeight,
          this.containerHeight * this.currentRatio
        )
      );

      // 下面板高度为容器高度减去上面板高度
      this.bottomPanelHeight = this.containerHeight - this.topPanelHeight;
      
      console.log(`初始化面板: ${this.uniqueId}, 高度: ${this.containerHeight}, 比例: ${this.currentRatio}`);
    },

    // 处理窗口大小变化
    handleResize() {
      // 仅在组件可见时处理
      if (!this.isVisible) return;
      
      // 使用防抖，减少事件触发频率
      if (this.resizeTimeout) {
        clearTimeout(this.resizeTimeout);
      }
      
      this.resizeTimeout = setTimeout(() => {
        this.handleContainerResize();
      }, 100);
    },

    // 开始拖动
    startDrag(event) {
      this.isDragging = true;
      event.preventDefault();
      event.stopPropagation();
      
      // 添加全局拖动样式
      document.body.style.cursor = 'row-resize';
      document.body.style.userSelect = 'none';
    },

    // 停止拖动
    stopDrag() {
      if (this.isDragging) {
        this.isDragging = false;
        
        // 恢复全局样式
        document.body.style.cursor = '';
        document.body.style.userSelect = '';
        
        // 保存当前比例
        this.currentRatio = this.topPanelHeight / this.containerHeight;
        
        // 触发一次终端重新适配事件
        this.$nextTick(() => {
          window.dispatchEvent(new Event('terminal-resize'));
        });
        
        console.log(`拖动结束: ${this.uniqueId}, 新比例: ${this.currentRatio}`);
      }
    },

    // 处理拖动过程 - 使用节流优化性能
    handleDrag(event) {
      if (!this.isDragging) return;
      
      // 使用requestAnimationFrame优化拖动性能
      if (this.dragAnimationFrame) {
        cancelAnimationFrame(this.dragAnimationFrame);
      }
      
      this.dragAnimationFrame = requestAnimationFrame(() => {
        const containerRect = this.$refs.splitterContainer.getBoundingClientRect();
        const newTopHeight = Math.max(
          this.minPanelHeight,
          Math.min(
            this.containerHeight - this.minPanelHeight,
            event.clientY - containerRect.top
          )
        );

        this.topPanelHeight = newTopHeight;
        this.bottomPanelHeight = this.containerHeight - newTopHeight;
      });
    }
  }
}
</script>

<style scoped>
.terminal-vsplitter {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
  box-sizing: border-box;
}

.panel {
  width: 100%;
  overflow: auto; /* 允许内容滚动 */
  position: relative;
  box-sizing: border-box;
}

.top-panel {
  border-bottom: none;
  min-height: 50px;
}

.bottom-panel {
  border-top: none;
  min-height: 50px;
}

.splitter-handle {
  width: 100%;
  height: 4px;
  background-color: #3a3a3a;
  cursor: row-resize;
  position: absolute;
  z-index: 9;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.2);
  left: 0;
  right: 0;
  pointer-events: auto;
  touch-action: none;
}

.splitter-handle:hover {
  background-color: #4a89dc;
  height: 4px;
}

.splitter-handle:active {
  background-color: #3a70c0;
  height: 4px;
}
</style>