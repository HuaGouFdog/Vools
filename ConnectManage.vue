<script setup lang="ts">
import { ref, reactive, computed, defineAsyncComponent } from 'vue';

// 使用异步组件加载CreateConnect，避免在初始化时就加载它
const CreateConnect = defineAsyncComponent(() =>
  import('./CreateConnect.vue')
);

// 控制创建连接弹窗的显示
const showCreateConnect = ref(false);

// 关闭连接管理窗口
const closeConnectManage = () => {
  // 实现关闭窗口的逻辑
  console.log('关闭连接管理窗口');
};

// 连接列表数据
const connections = reactive([
  { id: 1, name: '开发服务器A', host: '10.10.10.101', port: '22', username: 'admin', status: '默认' },
  { id: 2, name: 'arm编译服务器', host: '10.10.10.102', port: '22', username: 'admin', status: '默认' },
  { id: 3, name: '测试跳板机', host: 'jump.example.com', port: '50022', username: 'tester', status: '默认' },
  { id: 4, name: '生产环境跳板', host: 'prod-jump.example.com', port: '50022', username: 'operator', status: '默认' },
  { id: 5, name: '数据库服务器', host: '10.10.20.103', port: '22', username: 'admin', status: '默认' },
]);

// 搜索关键词
const searchKeyword = ref('');

// 过滤后的连接列表
const filteredConnections = computed(() => {
  if (!searchKeyword.value) return connections;
  const keyword = searchKeyword.value.toLowerCase();
  return connections.filter(conn =>
    conn.name.toLowerCase().includes(keyword) ||
    conn.host.toLowerCase().includes(keyword) ||
    conn.username.toLowerCase().includes(keyword)
  );
});

// 清空搜索
const clearSearch = () => {
  searchKeyword.value = '';
};

// 管理连接
const manageConnections = () => {
  // 实现管理连接的逻辑
  console.log('管理连接');
};

// 添加新连接
const addNewConnection = () => {
  // 使用setTimeout确保UI更新后再显示弹窗
  setTimeout(() => {
    showCreateConnect.value = true;
  }, 0);
};

// 关闭创建连接弹窗
const closeCreateConnect = () => {
  showCreateConnect.value = false;
};

// 保存新连接
const saveNewConnection = (connectionData) => {
  // 这里可以添加保存连接的逻辑
  console.log('保存新连接', connectionData);
  connections.push({
    id: connections.length + 1,
    name: connectionData.name || connectionData.host,
    host: connectionData.host,
    port: connectionData.port,
    username: connectionData.username,
    status: '默认'
  });
  closeCreateConnect();
};
</script>

<template>
  <div class="connect-manage">
    <!-- 搜索和添加区域 -->
    <div class="search-bar">
      <div class="search-input">
        <i class="search-icon">🔍</i>
        <input
          type="text"
          v-model="searchKeyword"
          placeholder="搜索名称，快速连接"
        />
      </div>
      <button class="add-btn" @click="addNewConnection">+</button>
    </div>

    <!-- 连接列表 -->
    <div class="connection-list">
      <div
        v-for="conn in filteredConnections"
        :key="conn.id"
        class="connection-item"
      >
        <div class="connection-status">
          <span class="status-dot"></span>
        </div>
        <div class="connection-name">{{ conn.name }}</div>
        <div class="connection-host">{{ conn.host }}</div>
        <div class="connection-port">{{ conn.port }}</div>
        <div class="connection-username">{{ conn.username }}</div>
        <div class="connection-action">
          <button class="connect-btn">默认</button>
        </div>
      </div>
    </div>

    <!-- 底部按钮区域 -->
    <div class="bottom-actions">
      <button class="manage-btn" @click="manageConnections">管理</button>
      <button class="clear-btn" @click="clearSearch">清空</button>
    </div>

    <!-- 创建连接弹窗 - 使用v-show代替v-if，避免频繁创建和销毁组件 -->
    <div v-show="showCreateConnect" class="modal-overlay" @click.self="closeCreateConnect">
      <div class="modal-container">
        <component
          :is="showCreateConnect ? CreateConnect : null"
          @save="saveNewConnection"
          @cancel="closeCreateConnect"
        />
      </div>
    </div>
  </div>
</template>

<style scoped>
.connect-manage {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  background-color: #1e2124;
  color: #ffffff;
  padding: 0;
  box-sizing: border-box;
  overflow: hidden; /* 防止内容溢出 */
  position: relative;
}

.search-bar {
  display: flex;
  margin: 16px;
  margin-bottom: 16px;
}

.search-input {
  flex: 1;
  position: relative;
  display: flex;
  align-items: center;
  background-color: #2c2f33;
  border-radius: 4px;
  padding: 0 10px;
}

.search-icon {
  margin-right: 8px;
  color: #7a7a7a;
}

.search-input input {
  flex: 1;
  background: transparent;
  border: none;
  color: #ffffff;
  padding: 10px 0;
  outline: none;
  width: 100%;
  font-size: 12px;
}

.add-btn {
  width: 36px;
  height: 36px;
  margin-left: 10px;
  background-color: #2c2f33;
  border: none;
  border-radius: 4px;
  color: #ffffff;
  font-size: 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.add-btn:hover {
  background-color: #3a3f45;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.add-btn:active {
  background-color: #252729;
  transform: translateY(1px);
}

.connection-list {
  flex: 1;
  overflow-y: auto;
  min-height: 0; /* 确保flex子元素可以正确收缩 */
  margin: 0 16px;
  margin-bottom: 0; /* 底部按钮区域有自己的边距 */
}

.connection-item {
  display: flex;
  align-items: center;
  padding: 8px;
  border-bottom: 1px solid #2c2f33;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.connection-item:hover {
  background-color: #2c2f33;
}

.connection-item:active {
  background-color: #3a3f45;
}

.connection-status {
  width: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.status-dot {
  width: 8px;
  height: 8px;
  background-color: #4caf50;
  border-radius: 50%;
}

.connection-name {
  width: 180px;
  padding: 0 10px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 12px;
}

.connection-host {
  width: 180px;
  padding: 0 10px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 12px;
}

.connection-port {
  width: 80px;
  padding: 0 10px;
  text-align: center;
  font-size: 12px;
}

.connection-username {
  width: 120px;
  padding: 0 10px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 12px;
}

.connection-action {
  flex: 1;
  display: flex;
  justify-content: flex-end;
}

.connect-btn {
  background-color: transparent;
  border: none;
  color: #7a7a7a;
  cursor: pointer;
  padding: 4px 8px;
  transition: all 0.2s ease;
  border-radius: 4px;
  font-size: 12px;
}

.connect-btn:hover {
  color: #ffffff;
  background-color: rgba(255, 255, 255, 0.1);
}

.connect-btn:active {
  background-color: rgba(255, 255, 255, 0.2);
}

.bottom-actions {
  display: flex;
  justify-content: space-between;
  padding: 10px 16px;
  border-top: 1px solid #2c2f33;
  background-color: #1e2124;
  z-index: 10;
}

.manage-btn, .clear-btn {
  padding: 6px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
  font-size: 12px;
}

.manage-btn {
  background-color: #4caf50;
  color: white;
}

.manage-btn:hover {
  background-color: #5dbb60;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.manage-btn:active {
  background-color: #3d9140;
  transform: translateY(1px);
}

.clear-btn {
  background-color: #2c2f33;
  color: white;
}

.clear-btn:hover {
  background-color: #3a3f45;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.clear-btn:active {
  background-color: #252729;
  transform: translateY(1px);
}

/* 弹窗样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-container {
  background-color: #1e2227;
  border-radius: 4px;
  width: 650px;
  height: 500px;
  max-width: 90%;
  max-height: 90%;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
}
</style>