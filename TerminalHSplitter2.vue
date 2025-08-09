<template>
  <div class="terminal-hsplitter2" ref="splitterContainer" :class="{ 'dragging': isDragging }">
    <div class="panel-container">
      <div class="panel left-panel" :style="{ width: leftPanelWidth + 'px' }">
        <CommandManage class="full-height" />
      </div>
      <div
        class="splitter-handle"
        @mousedown.stop="startDrag"
        :style="{ left: leftPanelWidth + 'px' }"
      ></div>
      <div class="panel right-panel" :style="{ width: rightPanelWidth + 'px' }">
        <slot name="right">
          <TerminalTextEdit class="full-height" />
        </slot>
      </div>
    </div>
  </div>
</template>

<script>
import { defineAsyncComponent, markRaw } from 'vue';

// 使用异步组件加载，减少初始加载时间
const CommandManage = defineAsyncComponent(() => import('./CommandManage.vue'));
const TerminalTextEdit = defineAsyncComponent(() => import('./TerminalTextEdit.vue'));

// 全局状态存储，使用静态对象而不是Map
const globalState = {
  // 默认比例 - 所有实例共享的默认值
  defaultRatio: 0.67,
  // 最后一次使用的比例 - 用于新创建的实例
  lastUsedRatio: 0.67
};

export default {
  name: 'TerminalHSplitter2',
  components: {
    CommandManage: markRaw(CommandManage),
    TerminalTextEdit: markRaw(TerminalTextEdit)
  },
  props: {
    // 唯一标识符，用于保存和恢复状态
    uniqueId: {
      type: String,
      default: () => `hsplitter2-${Date.now()}-${Math.floor(Math.random() * 1000)}`
    },
    // 初始分割比例，范围0-1，表示左面板所占比例
    initialRatio: {
      type: Number,
      default: 0.67 // 2:1的比例，左侧占2/3
    },
    // 最小面板宽度（像素）
    minPanelWidth: {
      type: Number,
      default: 150 // 确保命令分类能够完整显示
    }
  },
  data() {
    return {
      // 左面板宽度
      leftPanelWidth: 0,
      // 右面板宽度
      rightPanelWidth: 0,
      // 容器总宽度
      containerWidth: 0,
      // 拖动状态
      isDragging: false,
      // 当前分割比例 - 使用全局最后使用的比例或初始比例
      currentRatio: globalState.lastUsedRatio || this.initialRatio,
      // 组件是否已初始化
      isInitialized: false,
      // 组件是否可见
      isVisible: true,
      // ResizeObserver 实例
      resizeObserver: null,
      // 是否正在处理resize事件
      isResizing: false,
      // 上次resize时间戳
      lastResizeTime: 0
    }
  },
  created() {
    // 绑定事件处理函数到实例，避免内存泄漏
    this.boundHandleResize = this.debounce(this.handleResize.bind(this), 50);
    this.boundStopDrag = this.stopDrag.bind(this);
    this.boundHandleDrag = this.handleDrag.bind(this);
    
    console.log(`[TerminalHSplitter2] 创建: ${this.uniqueId}, 使用比例: ${this.currentRatio}`);
  },
  mounted() {
    console.log(`[TerminalHSplitter2] 挂载: ${this.uniqueId}`);
    
    // 创建 ResizeObserver 监听容器大小变化
    this.resizeObserver = new ResizeObserver(entries => {
      if (this.isVisible && !this.isResizing) {
        this.boundHandleResize();
      }
    });
    
    if (this.$refs.splitterContainer) {
      this.resizeObserver.observe(this.$refs.splitterContainer);
    }
    
    // 添加一个短暂延迟，确保DOM已完全渲染
    setTimeout(() => {
      this.initializePanels();
      this.isInitialized = true;
    }, 50);

    // 添加事件监听器
    window.addEventListener('resize', this.boundHandleResize);
    document.addEventListener('mouseup', this.boundStopDrag);
    document.addEventListener('mousemove', this.boundHandleDrag);
    
    // 监听可见性变化
    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        const wasVisible = this.isVisible;
        this.isVisible = entry.isIntersecting;
        
        if (!wasVisible && this.isVisible) {
          // 元素从不可见变为可见时重新计算布局
          console.log(`[TerminalHSplitter2] ${this.uniqueId} 变为可见，重新计算布局`);
          setTimeout(() => this.handleResize(), 50);
        }
      });
    }, { threshold: 0.1 });
    
    if (this.$refs.splitterContainer) {
      observer.observe(this.$refs.splitterContainer);
    }
    
    // 保存 observer 以便后续清理
    this._visibilityObserver = observer;
  },
  beforeUnmount() {
    console.log(`[TerminalHSplitter2] 卸载: ${this.uniqueId}`);
    
    // 移除事件监听器
    window.removeEventListener('resize', this.boundHandleResize);
    document.removeEventListener('mouseup', this.boundStopDrag);
    document.removeEventListener('mousemove', this.boundHandleDrag);
    
    // 清理 ResizeObserver
    if (this.resizeObserver) {
      this.resizeObserver.disconnect();
      this.resizeObserver = null;
    }
    
    // 清理可见性观察器
    if (this._visibilityObserver) {
      this._visibilityObserver.disconnect();
      this._visibilityObserver = null;
    }
    
    // 清理引用
    this.boundHandleResize = null;
    this.boundStopDrag = null;
    this.boundHandleDrag = null;
  },
  methods: {
    // 防抖函数
    debounce(func, wait) {
      let timeout;
      return function(...args) {
        const context = this;
        clearTimeout(timeout);
        timeout = setTimeout(() => func.apply(context, args), wait);
      };
    },
    
    // 初始化面板尺寸
    initializePanels() {
      // 确保容器元素存在
      if (!this.$refs.splitterContainer) {
        console.warn('[TerminalHSplitter2] 分割容器元素不存在，使用默认宽度');
        this.containerWidth = 800; // 提供默认宽度
      } else {
        this.containerWidth = this.$refs.splitterContainer.clientWidth || 800;
      }

      // 使用当前比例
      const ratio = this.currentRatio;
      
      // 确保左面板至少有最小宽度，且不超过容器宽度减去最小宽度
      this.leftPanelWidth = Math.max(
        this.minPanelWidth,
        Math.min(
          this.containerWidth - this.minPanelWidth,
          this.containerWidth * ratio
        )
      );

      // 右面板宽度为容器宽度减去左面板宽度
      this.rightPanelWidth = this.containerWidth - this.leftPanelWidth;

      console.log(`[TerminalHSplitter2] 初始化面板: ${this.uniqueId}`, {
        containerWidth: this.containerWidth,
        leftPanelWidth: this.leftPanelWidth,
        rightPanelWidth: this.rightPanelWidth,
        ratio: ratio
      });
    },

    // 处理窗口大小变化
    handleResize() {
      // 防止重入
      if (this.isResizing) return;
      this.isResizing = true;
      
      // 记录当前时间
      const now = Date.now();
      if (now - this.lastResizeTime < 50) {
        this.isResizing = false;
        return;
      }
      this.lastResizeTime = now;
      
      try {
        // 如果组件不可见或容器不存在，则跳过
        if (!this.isVisible || !this.$refs.splitterContainer) {
          return;
        }

        const oldWidth = this.containerWidth;
        const newWidth = this.$refs.splitterContainer.clientWidth;

        // 如果宽度没有变化，不需要重新计算
        if (oldWidth === newWidth && oldWidth !== 0 && this.isInitialized) {
          return;
        }

        this.containerWidth = newWidth;

        // 使用当前比例
        const ratio = this.currentRatio;

        // 保持分割比例不变
        this.leftPanelWidth = Math.max(
          this.minPanelWidth,
          Math.min(
            this.containerWidth - this.minPanelWidth,
            this.containerWidth * ratio
          )
        );

        this.rightPanelWidth = this.containerWidth - this.leftPanelWidth;
        
        console.log(`[TerminalHSplitter2] 调整大小: ${this.uniqueId}`, {
          containerWidth: this.containerWidth,
          leftPanelWidth: this.leftPanelWidth,
          rightPanelWidth: this.rightPanelWidth,
          ratio: ratio
        });
      } finally {
        this.isResizing = false;
      }
    },

    // 开始拖动
    startDrag(event) {
      this.isDragging = true;
      event.preventDefault();
      event.stopPropagation();

      // 添加拖动时的样式类，提高用户体验
      document.body.style.cursor = 'col-resize';
      document.body.style.userSelect = 'none';
      
      console.log(`[TerminalHSplitter2] 开始拖动: ${this.uniqueId}`);
    },

    // 停止拖动
    stopDrag() {
      if (this.isDragging) {
        // 恢复正常样式
        document.body.style.cursor = '';
        document.body.style.userSelect = '';
        
        // 更新当前比例
        this.currentRatio = this.leftPanelWidth / this.containerWidth;
        
        // 更新全局最后使用的比例
        globalState.lastUsedRatio = this.currentRatio;

        console.log(`[TerminalHSplitter2] 停止拖动: ${this.uniqueId}, 比例: ${this.currentRatio}`);
        
        this.isDragging = false;
      }
    },

    // 处理拖动过程 - 使用 requestAnimationFrame 优化性能
    handleDrag(event) {
      if (!this.isDragging) return;

      // 使用 requestAnimationFrame 优化性能
      requestAnimationFrame(() => {
        // 确保容器元素存在
        if (!this.$refs.splitterContainer) return;
        
        // 获取容器位置
        const containerRect = this.$refs.splitterContainer.getBoundingClientRect();
        
        // 计算新的左面板宽度
        const newLeftWidth = Math.max(
          this.minPanelWidth,
          Math.min(
            this.containerWidth - this.minPanelWidth,
            event.clientX - containerRect.left
          )
        );

        // 更新面板宽度
        this.leftPanelWidth = newLeftWidth;
        this.rightPanelWidth = this.containerWidth - newLeftWidth;
        
        // 更新当前比例
        this.currentRatio = this.leftPanelWidth / this.containerWidth;
      });
    }
  }
}
</script>

<style scoped>
.terminal-hsplitter2 {
  width: 100%;
  height: 100%;
  position: relative;
  background-color: #1e2124; /* 与终端背景一致 */
  overflow: hidden;
  box-sizing: border-box;
}

.panel-container {
  display: flex;
  flex-direction: row;
  width: 100%;
  height: 100%;
  position: relative;
  box-sizing: border-box;
}

.panel {
  height: 100%;
  overflow: hidden; /* 防止内容溢出 */
  box-sizing: border-box;
  position: relative;
}

.left-panel {
  border-right: none;
  position: relative;
  min-width: 150px; /* 确保左侧面板有足够的最小宽度 */
  background-color: #252526; /* 与命令管理背景一致 */
  flex-shrink: 0; /* 防止左侧面板被压缩 */
}

.right-panel {
  border-left: none;
  position: relative;
  flex: 1; /* 确保右侧面板填充剩余空间 */
  display: flex; /* 使用flex布局 */
  background-color: #1e2124; /* 与终端背景一致 */
}

.splitter-handle {
  width: 4px;
  height: 100%;
  background-color: #3a3a3a;
  cursor: col-resize;
  position: absolute;
  z-index: 10; /* 提高z-index确保可点击 */
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.2);
  margin-left: -2px; /* 向左偏移一半宽度，使分隔条居中 */
  pointer-events: auto; /* 确保可以接收鼠标事件 */
  top: 0;
  bottom: 0;
}

.splitter-handle:hover {
  background-color: #4a89dc;
}

.splitter-handle:active {
  background-color: #3a70c0;
}

/* 添加拖动时的样式 */
.terminal-hsplitter2.dragging .splitter-handle {
  background-color: #3a70c0;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.3);
}

.full-height {
  height: 100%;
  width: 100%;
  box-sizing: border-box;
  border: none;
  display: flex;
  flex-direction: column;
}
</style>