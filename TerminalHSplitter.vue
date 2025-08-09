<template>
  <div class="terminal-hsplitter" ref="splitterContainer">
    <div class="panel left-panel" :style="{ width: leftPanelWidth + 'px' }">
      <TerminalInfo />
    </div>
    <div
      class="splitter-handle"
      @mousedown="startDrag"
      :style="{ left: leftPanelWidth + 'px' }"
    ></div>
    <div class="panel right-panel" :style="{ width: rightPanelWidth + 'px' }">
      <TerminalVSplitter>
        <template #top>
          <slot name="vsplitter-top"></slot>
        </template>
        <template #bottom>
          <slot name="vsplitter-bottom"></slot>
        </template>
      </TerminalVSplitter>
    </div>
  </div>
</template>

<script>
import TerminalInfo from './TerminalInfo.vue';
import TerminalVSplitter from './TerminalVSplitter.vue';

export default {
  name: 'TerminalHSplitter',
  components: {
    TerminalInfo,
    TerminalVSplitter
  },
  props: {
    // 初始分割比例，范围0-1，表示左面板所占比例
    initialRatio: {
      type: Number,
      default: 0.3
    },
    // 最小面板宽度（像素）
    minPanelWidth: {
      type: Number,
      default: 200
    },
    // 最大面板宽度（像素）
    maxPanelWidth: {
      type: Number,
      default: 350
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
      isDragging: false
    }
  },
  mounted() {
    this.initializePanels()
    window.addEventListener('resize', this.handleResize)
    window.addEventListener('mouseup', this.stopDrag)
    window.addEventListener('mousemove', this.handleDrag)

    // 确保面板初始化后触发终端重新适配
    this.$nextTick(() => {
      // 使用多个延迟触发终端重新适配事件
      const delays = [50, 200, 500, 1000];
      delays.forEach(delay => {
        setTimeout(() => {
          window.dispatchEvent(new Event('terminal-resize'));
        }, delay);
      });
    })
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handleResize)
    window.removeEventListener('mouseup', this.stopDrag)
    window.removeEventListener('mousemove', this.handleDrag)
  },
  methods: {
    // 初始化面板尺寸
    initializePanels() {
      this.$nextTick(() => {
        this.containerWidth = this.$refs.splitterContainer.clientWidth
        // 将左侧面板宽度固定为200px，但不超过最大宽度
        this.leftPanelWidth = Math.min(200, this.maxPanelWidth)
        this.rightPanelWidth = this.containerWidth - this.leftPanelWidth
      })
    },

    // 处理窗口大小变化
    handleResize() {
      this.containerWidth = this.$refs.splitterContainer.clientWidth

      // 保持左侧面板宽度固定为200px，但不超过最大宽度
      this.leftPanelWidth = Math.min(200, this.maxPanelWidth)
      this.rightPanelWidth = this.containerWidth - this.leftPanelWidth
      
      // 触发终端重新适配事件
      this.$nextTick(() => {
        window.dispatchEvent(new Event('terminal-resize'));
      });
    },

    // 开始拖动
    startDrag(event) {
      this.isDragging = true
      event.preventDefault()
    },

    // 停止拖动
    stopDrag() {
      this.isDragging = false
    },

    // 处理拖动过程
    handleDrag(event) {
      if (!this.isDragging) return

      const containerRect = this.$refs.splitterContainer.getBoundingClientRect()
      const newLeftWidth = Math.max(
        this.minPanelWidth,
        Math.min(
          Math.min(this.containerWidth - this.minPanelWidth, this.maxPanelWidth),
          event.clientX - containerRect.left
        )
      )

      this.leftPanelWidth = newLeftWidth
      this.rightPanelWidth = this.containerWidth - this.leftPanelWidth

      // 触发分割比例变化事件
      this.$emit('split-change', {
        ratio: this.leftPanelWidth / this.containerWidth,
        leftWidth: this.leftPanelWidth,
        rightWidth: this.rightPanelWidth
      })
      
      // 触发终端重新适配事件
      this.$nextTick(() => {
        window.dispatchEvent(new Event('terminal-resize'));
      });
    }
  }
}
</script>

<style scoped>
.terminal-hsplitter {
  display: flex;
  flex-direction: row;
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
  /* 确保内容不会溢出 */
  contain: layout paint;
}

.panel {
  height: 100%;
  overflow: auto;
}

.left-panel {
  border-right: none;
}

.right-panel {
  border-left: none;
}

.splitter-handle {
  width: 4px;
  height: 100%;
  background-color: #3a3a3a;
  cursor: col-resize;
  position: absolute;
  z-index: 10; /* 降低z-index，确保不会超出其容器 */
  transition: background-color 0.2s ease;
  box-shadow: 0 0 3px rgba(0, 0, 0, 0.2);
  /* 确保分隔条不会超出其容器 */
  top: 0;
  bottom: 0;
  pointer-events: auto;
}

.splitter-handle:hover {
  background-color: #4a89dc;
  width: 4px;
}

.splitter-handle:active {
  background-color: #3a70c0;
  width: 4px;
}
</style>
