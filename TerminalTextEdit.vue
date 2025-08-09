<template>
  <div class="terminal-text-edit">
    <div class="text-edit-header">
      <div class="header-title">命令编辑器</div>
    </div>
    <div class="text-edit-content">
      <textarea
        v-model="content"
        class="editor-textarea"
        placeholder="在此输入命令内容..."
        @input="handleInput"
        ref="editorTextarea"
      ></textarea>
    </div>
    <div class="text-edit-footer">
      <div class="footer-info">
        <span class="line-count">行数: {{ lineCount }}</span>
        <span class="char-count">字符数: {{ charCount }}</span>
      </div>
      <button class="run-btn" @click="runCommand">
        <i class="fas fa-play"></i> 执行
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'TerminalTextEdit',
  props: {
    initialContent: {
      type: String,
      default: ''
    },
    commandName: {
      type: String,
      default: '新建命令'
    }
  },
  data() {
    return {
      content: this.initialContent,
      savedContent: this.initialContent,
      lineCount: 1,
      charCount: 0
    };
  },
  watch: {
    initialContent(newValue) {
      this.content = newValue;
      this.savedContent = newValue;
      this.updateCounts();
    }
  },
  mounted() {
    this.updateCounts();
    // 设置焦点
    this.$nextTick(() => {
      this.$refs.editorTextarea.focus();
    });
  },
  methods: {
    handleInput() {
      this.updateCounts();
    },
    updateCounts() {
      this.charCount = this.content.length;
      this.lineCount = this.content ? this.content.split('\n').length : 1;
    },
    runCommand() {
      // 自动保存内容
      this.savedContent = this.content;
      this.$emit('run', this.content);
    },
    // 公开方法，供父组件调用
    setContent(newContent) {
      this.content = newContent;
      this.updateCounts();
    },
    getContent() {
      return this.content;
    }
  }
};
</script>

<style scoped>
.terminal-text-edit {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  min-width: 100%;
  max-width: 100%;
  background-color: #3a3a3a; /* 修改为与左侧组件相同的背景色 */
  color: #e0e0e0;
  font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  border: none;
  box-sizing: border-box;
  overflow: hidden; /* 防止内容溢出 */
  flex: 1; /* 确保填充所有可用空间 */
}

.text-edit-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  /* 与左侧组件保持一致的内边距 */
  background-color: #3a3a3a; /* 与左侧组件相同的背景色 */
  border-bottom: 1px solid #555; /* 与左侧组件相同的边框颜色 */
  height: 36px; /* 与左侧组件保持一致的高度 */
  box-sizing: border-box;
  width: 100%;
  margin-top: 3px; /* 增加与上方分隔条的间距，与左侧组件一致 */
  /* 将下边距增加1px，使线条向下移动1px */
  padding: 5px 5px 5px;
}

.header-title {
  padding: 0 0 0 6px;
  font-family: Arial, sans-serif;
  font-size: 12px; /* 将文字大小改为12px */
  font-weight: bold;
  color: #e0e0e0;
}

.text-edit-content {
  font-family: Arial, sans-serif;
  flex: 1;
  padding: 0;
  overflow: hidden;
  position: relative;
  width: 100%;
  display: flex; /* 使用flex布局 */
}

.editor-textarea {
  font-family: Arial, sans-serif;
  width: 100%;
  height: 100%;
  padding: 10px;
  background-color: #3a3a3a; /* 与左侧组件相同的背景色 */
  color: #e0e0e0;
  border: none;
  resize: none;
  font-size: 14px;
  line-height: 1.5;
  box-sizing: border-box;
  outline: none;
  flex: 1; /* 确保填充所有可用空间 */
}

.editor-textarea:focus {
  outline: none;
}

.text-edit-footer {
  display: flex;
  justify-content: space-between; /* 左右两端对齐 */
  align-items: center;
  padding: 6px 8px; /* 与左侧组件保持一致的内边距 */
  background-color: #333333; /* 与左侧组件命令执行区域相同的背景色 */
  border-top: 1px solid #555; /* 与左侧组件相同的边框颜色 */
  height: 28px;
  font-size: 12px;
  color: #999;
  width: 100%;
  position: relative; /* 添加相对定位 */
}

.footer-info {
  font-family: Arial, sans-serif;
  display: flex;
  gap: 12px;
}

.run-btn {
  padding: 4px 10px;
  background-color: #1890ff; /* 与左侧组件按钮相同的颜色 */
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 12px;
  height: 28px;
  line-height: 16px;
  transition: background-color 0.2s ease;
  box-shadow: 0 2px 0 rgba(0, 0, 0, 0.2);
  user-select: none; /* 禁止选中文本 */
  display: flex;
  align-items: center;
  gap: 4px;
  position: absolute; /* 使用绝对定位 */
  right: 30px; /* 距离右侧20px，避免被挡住 */
  top: 50%;
  transform: translateY(-50%); /* 垂直居中 */
}

.run-btn:hover {
  background-color: #40a9ff;
}

.run-btn:active {
  background-color: #096dd9;
}

/* 自定义滚动条样式 */
.editor-textarea::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.editor-textarea::-webkit-scrollbar-track {
  background: #444;
}

.editor-textarea::-webkit-scrollbar-thumb {
  background: #555;
  border-radius: 4px;
}

.editor-textarea::-webkit-scrollbar-thumb:hover {
  background: #666;
}
</style>