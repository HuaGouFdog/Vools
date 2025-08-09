<template>
  <div class="command-manage">
    <!-- 命令分类 -->
    <div class="command-categories">
      <div
        v-for="(category, index) in categories"
        :key="index"
        class="category-item"
        :class="{ active: currentCategory === category.value }"
        @click="selectCategory(category.value)"
      >
        <i :class="category.icon"></i> {{ category.label }}
      </div>
    </div>

    <!-- 命令列表 -->
    <div class="command-list">
      <div class="command-grid">
        <div
          v-for="(command, index) in filteredCommands"
          :key="index"
          class="command-item"
          @click="selectCommand(command)"
        >
          {{ command.name }} <i class="fas fa-cog"></i>
        </div>
      </div>
    </div>

    <!-- 命令执行区域 -->
    <div class="command-execution">
      <div class="command-content">
        <div class="command-name">{{ currentCommand.name }}:</div>
        <input type="text" v-model="commandInput" class="command-input" />
        <button class="edit-btn" @click="editCommand">编辑</button>
      </div>
    </div>

    <!-- 编辑命令弹窗 -->
    <div class="command-modal" v-if="showModal">
      <div class="modal-content">
        <h3>编辑命令</h3>
        <div class="form-group">
          <label>命令名称</label>
          <input type="text" v-model="editingCommand.name" />
        </div>
        <div class="form-group">
          <label>命令内容</label>
          <textarea v-model="editingCommand.command"></textarea>
        </div>
        <div class="form-group">
          <label>分类</label>
          <select v-model="editingCommand.category">
            <option v-for="(category, index) in categories" :key="index" :value="category.value">
              {{ category.label }}
            </option>
          </select>
        </div>
        <div class="modal-actions">
          <button @click="saveCommand">保存</button>
          <button @click="cancelEdit">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CommandManage',
  data() {
    return {
      currentCategory: 'all',
      currentCommand: { name: '执行命令', command: '示例命令内容' },
      commandInput: '示例命令内容',
      showModal: false,
      editingCommand: { name: '', command: '', category: '' },
      categories: [
        { label: '默认分类', value: 'all', icon: 'fas fa-list' },
        { label: 'java', value: 'java', icon: 'fab fa-java' },
        { label: 'minic', value: 'minic', icon: 'fas fa-server' },
        { label: '打包/运维', value: 'ops', icon: 'fas fa-archive' },
        { label: 'docker', value: 'docker', icon: 'fab fa-docker' },
        { label: '抓包', value: 'capture', icon: 'fas fa-network-wired' },
        { label: 'git', value: 'git', icon: 'fab fa-git-alt' },
        { label: '数据库', value: 'database', icon: 'fas fa-database' }
      ],
      commands: [
        { name: '查询服务状态', command: 'service status', category: 'minic' },
        { name: '连接主数据库', command: 'db connect main', category: 'database' },
        { name: '停止自动服务', command: 'service stop auto', category: 'ops' },
        { name: '查看配置目录', command: 'ls -la /etc/config', category: 'all' },
        { name: '设置环境变量', command: 'export VAR=value', category: 'all' },
        { name: '查看环境变量', command: 'env | grep VAR', category: 'all' },
        { name: '网络抓包', command: 'tcpdump -i eth0', category: 'capture' },
        { name: '进入应用目录', command: 'cd /app/main', category: 'minic' },
        { name: '查看应用日志', command: 'tail -f /var/log/app.log', category: 'minic' },
        { name: '抓取HTTP包', command: 'tcpdump port 80', category: 'capture' },
        { name: '文件加密', command: 'openssl enc -aes-256-cbc -in file', category: 'all' },
        { name: '安装GCC 5.4', command: 'apt install gcc-5.4', category: 'ops' },
        { name: 'ARM交叉编译', command: 'arm-linux-gnueabi-gcc', category: 'ops' },
        { name: '抓取HTTPS包', command: 'tcpdump port 443', category: 'capture' },
        { name: '抓取DNS包', command: 'tcpdump port 53', category: 'capture' },
        { name: '安装GCC 4.8.5', command: 'apt install gcc-4.8.5', category: 'ops' },
        { name: '更新系统GCC', command: 'apt upgrade gcc', category: 'ops' },
        { name: '配置GCC路径', command: 'export PATH=/usr/local/gcc/bin:$PATH', category: 'ops' },
        { name: '传输文件到服务器', command: 'scp file user@server:/path', category: 'ops' },
        { name: '查看服务器密码', command: 'cat ~/.ssh/config', category: 'ops' },
        { name: '切换应用版本', command: 'app-switch v2.1', category: 'minic' },
        { name: '查看SSH密钥', command: 'cat ~/.ssh/id_rsa.pub', category: 'ops' },
        { name: '更新应用配置', command: 'app-config update', category: 'minic' },
        { name: '远程复制应用', command: 'scp -r app/ server:/opt/', category: 'ops' },
        { name: '生成SSH密钥', command: 'ssh-keygen -t rsa', category: 'ops' }
      ]
    };
  },
  computed: {
    filteredCommands() {
      if (this.currentCategory === 'all') {
        return this.commands;
      } else {
        return this.commands.filter(cmd => cmd.category === this.currentCategory);
      }
    }
  },
  methods: {
    selectCategory(category) {
      this.currentCategory = category;
    },
    selectCommand(command) {
      this.currentCommand = command;
      this.commandInput = command.command;
    },
    editCommand() {
      this.editingCommand = { ...this.currentCommand };
      this.showModal = true;
    },
    saveCommand() {
      // 查找当前命令在数组中的索引
      const index = this.commands.findIndex(cmd => cmd.name === this.currentCommand.name);
      if (index !== -1) {
        // 更新命令
        this.commands[index] = { ...this.editingCommand };
        this.currentCommand = { ...this.editingCommand };
        this.commandInput = this.editingCommand.command;
      } else {
        // 添加新命令
        this.commands.push({ ...this.editingCommand });
        this.currentCommand = { ...this.editingCommand };
        this.commandInput = this.editingCommand.command;
      }
      this.showModal = false;
    },
    cancelEdit() {
      this.showModal = false;
    },
    executeCommand() {
      // 这里可以添加执行命令的逻辑
      console.log('执行命令:', this.commandInput);
      // 可能需要调用父组件的方法或者发送事件
      this.$emit('execute-command', this.commandInput);
    }
  }
};
</script>

<style scoped>
.command-manage {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  font-size: 12px;
  border: none;
  box-sizing: border-box;
  background-color: #3a3a3a; /* 改为灰色背景 */
  overflow: hidden;
  position: relative;
  z-index: 1;
  color: #e0e0e0; /* 文字颜色改为浅色 */
}

.command-categories {
  display: flex;
  flex-wrap: wrap;
  padding: 6px;
  background-color: #3a3a3a; /* 改为灰色背景 */
  border-bottom: 1px solid #555; /* 边框颜色调整 */
  gap: 6px;
  position: relative;
  z-index: 1;
  margin-top: 2px; /* 增加与上方分隔条的间距 */
}

.category-item {
  padding: 4px 8px;
  border: 1px solid #555;
  border-radius: 3px;
  cursor: pointer;
  background-color: #4a4a4a; /* 深灰色背景 */
  height: 24px;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: center; /* 添加水平居中 */
  font-size: 12px;
  transition: all 0.2s ease;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
  color: #e0e0e0; /* 浅色文字 */
  user-select: none; /* 禁止选中文本 */
  line-height: 1; /* 确保文本垂直居中 */
}

.category-item.active {
  background-color: #1890ff;
  border-color: #1890ff;
  color: white;
  box-shadow: 0 1px 3px rgba(24, 144, 255, 0.3);
}

.category-item:hover:not(.active) {
  border-color: #1890ff;
  color: #1890ff;
  background-color: #404040; /* 悬停时稍微亮一点 */
}

.category-item i {
  margin-right: 4px;
  font-size: 12px;
}

.command-list {
  padding: 6px;
  overflow-y: auto;
  flex: 1;
  background-color: #3a3a3a; /* 改为灰色背景 */
  scrollbar-width: thin; /* Firefox */
  scrollbar-color: #555 #2a2a2a; /* Firefox */
}

.command-grid {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  align-content: flex-start;
  gap: 6px;
}

.command-item {
  padding: 4px 8px;
  border: 1px solid #555;
  border-radius: 3px;
  cursor: pointer;
  background-color: #4a4a4a; /* 深灰色背景 */
  color: #e0e0e0; /* 浅色文字 */
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 24px;
  box-sizing: border-box;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-size: 12px;
  user-select: none; /* 禁止选中文本 */
  transition: all 0.2s ease;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.2);
  width: auto; /* 根据内容自适应宽度 */
  min-width: 0; /* 移除最小宽度限制 */
}

.command-item:hover {
  background-color: #555555; /* 悬停时稍微亮一点 */
  border-color: #1890ff;
  color: #1890ff;
  box-shadow: 0 1px 3px rgba(24, 144, 255, 0.2);
}

.command-item i {
  margin-left: 6px;
  color: #999;
  font-size: 12px;
}

.command-execution {
  padding: 6px 8px;
  background-color: #333333; /* 深灰色背景 */
  border-top: 1px solid #555; /* 边框颜色调整 */
  margin-right: 0;
}

.command-name {
  font-weight: bold;
  padding: 0 4px;
  margin-right: 6px;
  white-space: nowrap;
  font-size: 12px;
  line-height: 16px;
  display: flex;
  align-items: center;
  color: #e0e0e0; /* 浅色文字 */
  user-select: none; /* 禁止选中文本 */
}

.command-content {
  display: flex;
  align-items: center;
  height: 28px;
}

.command-input {
  flex: 1;
  padding: 4px 8px;
  border: 1px solid #555;
  border-radius: 3px;
  margin-right: 8px;
  font-size: 12px;
  height: 28px;
  line-height: 16px;
  box-sizing: border-box;
  background-color: #4a4a4a; /* 深灰色背景 */
  color: #e0e0e0; /* 浅色文字 */
  transition: border-color 0.2s ease;
}

.command-input:focus {
  border-color: #1890ff;
  outline: none;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
  background-color: #555; /* 聚焦时稍微亮一点 */
}

.edit-btn {
  padding: 4px 10px;
  background-color: #1890ff;
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
}

.edit-btn:hover {
  background-color: #40a9ff;
}

.edit-btn:active {
  background-color: #096dd9;
}

.command-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7); /* 更暗的背景 */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1001; /* 设置为1001，确保显示在所有分隔条上面 */
}

.modal-content {
  background-color: #333333; /* 深灰色背景 */
  padding: 20px;
  border-radius: 6px;
  width: 500px;
  max-width: 90%;
  font-size: 12px;
  color: #e0e0e0; /* 浅色文字 */
  box-shadow: 0 3px 6px -4px rgba(0, 0, 0, 0.3), 0 6px 16px 0 rgba(0, 0, 0, 0.2), 0 9px 28px 8px rgba(0, 0, 0, 0.15);
  border: 1px solid #555;
  position: relative; /* 添加相对定位 */
  z-index: 1002; /* 确保内容在弹窗背景上层 */
}

.modal-content h3 {
  font-size: 16px;
  margin-top: 0;
  margin-bottom: 16px;
  color: #e0e0e0; /* 浅色文字 */
  user-select: none; /* 禁止选中文本 */
}

.form-group {
  margin-bottom: 16px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  font-weight: bold;
  font-size: 12px;
  color: #e0e0e0; /* 浅色文字 */
  user-select: none; /* 禁止选中文本 */
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 8px;
  border: 1px solid #555;
  border-radius: 3px;
  font-size: 12px;
  transition: all 0.3s;
  background-color: #4a4a4a; /* 深灰色背景 */
  color: #e0e0e0; /* 浅色文字 */
}

.form-group input:focus,
.form-group textarea:focus,
.form-group select:focus {
  border-color: #1890ff;
  outline: none;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
  background-color: #555; /* 聚焦时稍微亮一点 */
}

.form-group textarea {
  height: 100px;
  resize: vertical;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
}

.modal-actions button {
  padding: 6px 16px;
  margin-left: 8px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 12px;
  height: 28px;
  transition: all 0.3s;
}

.modal-actions button:first-child {
  background-color: #1890ff;
  color: white;
  box-shadow: 0 2px 0 rgba(0, 0, 0, 0.045);
}

.modal-actions button:first-child:hover {
  background-color: #40a9ff;
}

.modal-actions button:last-child {
  background-color: #f5f5f5;
  border: 1px solid #d9d9d9;
  color: #333;
}

.modal-actions button:last-child:hover {
  border-color: #1890ff;
  color: #1890ff;
}
</style>