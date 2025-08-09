<script setup lang="ts">
import { ref, onMounted } from 'vue';

// 模拟服务器信息数据，实际项目中会通过API获取
const serverInfo = ref({
  ip: '192.168.6.90',
  uptime: 73, // 运行天数
  load: ['1.55', '1.43', '1.35'], // 负载
  cpu: {
    usage: 2, // CPU使用率百分比
  },
  memory: {
    usage: 80, // 内存使用率百分比
    used: '101.6G',
    total: '126.2G'
  },
  swap: {
    usage: 0, // 交换分区使用率百分比
    used: '0',
    total: '0'
  },
  syncStatus: true, // 同步状态
  processes: [
    { memory: '60.2G', cpu: '110.3', command: 'qemu-kvm' },
    { memory: '10.5G', cpu: '1', command: 'qemu-kvm' },
    { memory: '12.6M', cpu: '0.7', command: 'sshd' },
    { memory: '8.7M', cpu: '0.7', command: 'top' }
  ],
  network: {
    upload: '71K',
    download: '5K',
    device: 'enp125s0f0',
    devices: ['enp125s0f0', 'eth0', 'wlan0', 'lo'],
    history: [76, 53, 26, 0, 0, 0],
    latency: ['0ms', '0', '0'],
    target: '本机'
  },
  disks: [
    { path: '/dev', available: '63.1G', total: '63.1G' },
    { path: '/dev/shm', available: '63.1G', total: '63.1G' },
    { path: '/run', available: '58.9G', total: '63.1G' },
    { path: '/sys/fs/cgroup', available: '63.1G', total: '63.1G' },
    { path: '/', available: '111.5G', total: '465.4G', highlight: true },
    { path: '/tmp', available: '63.1G', total: '63.1G' },
    { path: '/data', available: '1.4T', total: '2.5T' },
    { path: '/boot', available: '855M', total: '1014M' },
    { path: '/boot/efi', available: '592M', total: '598M' },
    { path: '/data/docker/overlay2/f0c...', available: '1.4T', total: '2.5T' },
    { path: '/data/docker/overlay2/f9b...', available: '1.4T', total: '2.5T' },
    { path: '/run/user/0', available: '12.6G', total: '12.6G' },
    { path: '/data/docker/overlay2/7b...', available: '1.4T', total: '2.5T' },
    { path: '/data/docker/overlay2/86...', available: '1.4T', total: '2.5T' },
    { path: '/data/docker/overlay2/3a...', available: '1.4T', total: '2.5T' },
    { path: '/data/docker/overlay2/ad...', available: '1.4T', total: '2.5T' },
    { path: '/data/docker/overlay2/fe5...', available: '1.4T', total: '2.5T' }
  ]
});

// 复制IP地址到剪贴板
const copyIP = () => {
  navigator.clipboard.writeText(serverInfo.value.ip)
    .then(() => {
      console.log('IP地址已复制到剪贴板');
      // 这里可以添加一个提示，告诉用户复制成功
    })
    .catch(err => {
      console.error('复制失败:', err);
    });
};

// 网络设备下拉框状态
const showNetworkDevices = ref(false);

// 切换网络设备
const selectNetworkDevice = (device: string) => {
  serverInfo.value.network.device = device;
  showNetworkDevices.value = false;
};

// 切换下拉框显示状态
const toggleNetworkDeviceDropdown = () => {
  showNetworkDevices.value = !showNetworkDevices.value;
};

// 在实际项目中，可以添加一个函数来从后端获取服务器信息
// const fetchServerInfo = async () => {
//   try {
//     const response = await fetch('/api/server-info');
//     const data = await response.json();
//     serverInfo.value = data;
//   } catch (error) {
//     console.error('获取服务器信息失败:', error);
//   }
// };

// onMounted(() => {
//   fetchServerInfo();
//   // 可以设置定时器定期更新服务器信息
//   // setInterval(fetchServerInfo, 30000); // 每30秒更新一次
// });
</script>

<template>
  <div class="terminal-info">
    <!-- 同步状态 -->
    <div class="sync-status">
      <span>同步状态</span>
      <div class="status-indicator" :class="{ active: serverInfo.syncStatus }"></div>
    </div>

    <!-- IP地址 -->
    <div class="info-row ip-row">
      <span class="ip">IP {{ serverInfo.ip }}</span>
      <button class="copy-btn" @click="copyIP">复制</button>
    </div>

    <!-- 运行时间 -->
    <div class="info-row">
      <span>运行 {{ serverInfo.uptime }} 天</span>
    </div>

    <!-- 负载 -->
    <div class="info-row">
      <span>负载 {{ serverInfo.load.join(', ') }}</span>
    </div>

    <!-- CPU使用率 -->
    <div class="info-row">
      <span>CPU</span>
      <div class="progress-bar">
        <div class="progress" :style="{ width: serverInfo.cpu.usage + '%', backgroundColor: '#4CAF50' }"></div>
        <span class="progress-text">{{ serverInfo.cpu.usage }}%</span>
      </div>
    </div>

    <!-- 内存使用率 -->
    <div class="info-row">
      <span>内存</span>
      <div class="progress-bar">
        <div class="progress" :style="{ width: serverInfo.memory.usage + '%', backgroundColor: '#FF9800' }"></div>
        <span class="progress-text">{{ serverInfo.memory.usage }}% {{ serverInfo.memory.used }}/{{ serverInfo.memory.total }}</span>
      </div>
    </div>

    <!-- 交换分区使用率 -->
    <div class="info-row">
      <span>交换</span>
      <div class="progress-bar">
        <div class="progress" :style="{ width: serverInfo.swap.usage + '%', backgroundColor: '#2196F3' }"></div>
        <span class="progress-text">{{ serverInfo.swap.usage }}% {{ serverInfo.swap.used }}/{{ serverInfo.swap.total }}</span>
      </div>
    </div>

    <!-- 进程信息 -->
    <div class="section-title">进程信息</div>
    <div class="process-table">
      <div class="process-header">
        <div class="header-cell memory">内存</div>
        <div class="header-cell cpu">处理器</div>
        <div class="header-cell command">命令</div>
      </div>
      <div class="process-row" v-for="(process, index) in serverInfo.processes" :key="index">
        <div class="cell memory">{{ process.memory }}</div>
        <div class="cell cpu">{{ process.cpu }}</div>
        <div class="cell command">{{ process.command }}</div>
      </div>
    </div>

    <!-- 网络流量 -->
    <div class="network-info">
      <div class="network-stats">
        <div class="network-stat upload">
          <span class="arrow">↑</span> {{ serverInfo.network.upload }}
        </div>
        <div class="network-stat download">
          <span class="arrow">↓</span> {{ serverInfo.network.download }}
        </div>
        <div class="network-device-dropdown">
          <div class="network-device" @click="toggleNetworkDeviceDropdown">
            {{ serverInfo.network.device }} <span class="dropdown">▼</span>
          </div>
          <div class="dropdown-menu" v-if="showNetworkDevices">
            <div
              v-for="device in serverInfo.network.devices"
              :key="device"
              class="dropdown-item"
              @click="selectNetworkDevice(device)"
            >
              {{ device }}
            </div>
          </div>
        </div>
      </div>

      <div class="network-chart">
        <div class="chart-bars">
          <div class="chart-bar" v-for="(value, index) in serverInfo.network.history" :key="index" :style="{ height: value + 'px' }"></div>
        </div>
      </div>

      <div class="network-latency">
        <div v-for="(latency, index) in serverInfo.network.latency" :key="index">{{ latency }}</div>
        <div class="network-target">{{ serverInfo.network.target }}</div>
      </div>
    </div>

    <!-- 磁盘使用情况 -->
    <div class="section-title">磁盘使用情况</div>
    <div class="disk-table">
      <div class="disk-header">
        <div class="disk-cell path">路径</div>
        <div class="disk-cell space">可用/大小</div>
      </div>
      <div class="disk-rows">
        <div class="disk-row" v-for="(disk, index) in serverInfo.disks" :key="index" :class="{ highlight: disk.highlight }">
          <div class="disk-cell path">{{ disk.path }}</div>
          <div class="disk-cell space" :class="{ highlight: disk.highlight }">{{ disk.available }}/{{ disk.total }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.terminal-info {
  font-family: Arial, sans-serif;
  padding: 8px;
  background-color: #2d2d2d;
  border-radius: 0;
  width: 100%;
  font-size: 12px;
  height: 100%;
  color: #e0e0e0;
  box-sizing: border-box;
  overflow-y: auto;
}

.sync-status {
  font-size: 12px;
  display: flex;
  align-items: center;
  margin-bottom: 8px;
  color: #e0e0e0;
}

.status-indicator {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  background-color: #ccc;
  margin-left: 8px;
}

.status-indicator.active {
  background-color: #4CAF50;
}

.info-row {
  font-size: 12px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.ip-row {
  margin-bottom: 12px;
}

.ip {
  font-weight: bold;
}

.copy-btn {
  background-color: transparent;
  border: none;
  color: #a0a0a0;
  cursor: pointer;
  padding: 2px 6px;
  font-size: 11px;
}

.copy-btn:hover {
  color: #ffffff;
  text-decoration: underline;
}

.progress-bar {
  flex: 1;
  height: 16px;
  background-color: #444444;
  border-radius: 3px;
  margin-left: 8px;
  position: relative;
  overflow: hidden;
}

.progress {
  height: 100%;
  transition: width 0.3s ease;
}

.progress-text {
  position: absolute;
  right: 5px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 11px;
  color: #ffffff;
}

/* 进程表格样式 */
.section-title {
  font-weight: bold;
  margin: 12px 0 8px 0;
  border-bottom: 1px solid #555;
  padding-bottom: 4px;
  font-size: 13px;
  color: #ffffff;
}

.process-table {
  border: 1px solid #555;
  border-radius: 3px;
  overflow: hidden;
  margin-bottom: 12px;
  font-size: 11px;
}

.process-header {
  display: flex;
  background-color: #4a89dc;
  color: white;
  font-weight: bold;
}

.header-cell, .cell {
  padding: 6px;
  text-align: left;
}

.memory {
  width: 25%;
}

.cpu {
  width: 25%;
  text-align: center;
}

.command {
  width: 50%;
}

.process-header .cpu {
  text-align: center;
}

.process-row .cell.cpu {
  text-align: center;
}

.process-row {
  display: flex;
  border-bottom: 1px solid #eee;
}

.process-row:last-child {
  border-bottom: none;
}

.process-row:nth-child(even) {
  background-color: #3a3a3a;
}

/* 网络流量样式 */
.network-info {
  border: 1px solid #555;
  border-radius: 3px;
  padding: 8px;
  margin: 12px 0;
  font-size: 11px;
}

.network-stats {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
}

.network-stat {
  display: flex;
  align-items: center;
}

.arrow {
  margin-right: 4px;
  font-weight: bold;
}

.upload .arrow {
  color: #e74c3c;
}

.download .arrow {
  color: #2ecc71;
}

.network-device-dropdown {
  position: relative;
}

.network-device {
  font-weight: bold;
  cursor: pointer;
  padding: 2px 4px;
  border-radius: 3px;
}

.network-device:hover {
  background-color: #444444;
}

.dropdown {
  font-size: 9px;
  margin-left: 4px;
}

.dropdown-menu {
  position: absolute;
  top: 100%;
  right: 0;
  background-color: #333333;
  border: 1px solid #555;
  border-radius: 3px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.3);
  z-index: 10;
  min-width: 100px;
  font-size: 11px;
}

.dropdown-item {
  padding: 6px 10px;
  cursor: pointer;
}

.dropdown-item:hover {
  background-color: #444444;
}

.network-chart {
  height: 60px;
  border-top: 1px solid #555;
  border-bottom: 1px solid #555;
  padding: 8px 0;
  margin: 8px 0;
}

.chart-bars {
  display: flex;
  justify-content: space-around;
  align-items: flex-end;
  height: 100%;
}

.chart-bar {
  width: 8px;
  background-color: #ff9800;
  border-radius: 2px 2px 0 0;
}

.network-latency {
  display: flex;
  justify-content: space-between;
  font-size: 10px;
}

.network-target {
  font-weight: bold;
}

/* 磁盘使用情况样式 */
.disk-table {
  border: 1px solid #555;
  border-radius: 3px;
  overflow: hidden;
  margin-bottom: 12px;
  font-size: 11px;
}

.disk-header {
  display: flex;
  background-color: #4a89dc;
  color: white;
  font-weight: bold;
}

.disk-cell {
  padding: 4px;
  text-align: left;
}

.disk-cell.path {
  width: 60%;
}

.disk-cell.space {
  width: 40%;
}

.disk-rows {
  max-height: none;
  overflow-y: auto;
}

.disk-row {
  display: flex;
  border-bottom: 1px solid #555;
}

.disk-row:last-child {
  border-bottom: none;
}

.disk-row:nth-child(even) {
  background-color: #3a3a3a;
}

.disk-cell.highlight, .disk-row.highlight {
  color: #64b5f6;
  font-weight: bold;
}
</style>