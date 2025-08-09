<script setup lang="ts">
import { ref, defineEmits } from 'vue';

const emit = defineEmits(['save', 'cancel']);

// 定义连接类型
const connectionTypes = ['ssh', 'windows', 'zookeeper', 'database', 'redis'];
const activeType = ref('ssh');

// 表单数据
const formData = ref({
  name: '',
  group: '',
  remark: '',
  host: '127.0.0.1',
  port: '22',
  username: 'root',
  password: '',
  rememberPassword: true
});

// 切换连接类型
const switchType = (type) => {
  activeType.value = type;
  // 根据不同类型设置默认端口
  switch(type) {
    case 'ssh':
      formData.value.port = '22';
      break;
    case 'windows':
      formData.value.port = '3389';
      break;
    case 'zookeeper':
      formData.value.port = '2181';
      break;
    case 'database':
      formData.value.port = '3306';
      break;
    case 'redis':
      formData.value.port = '6379';
      break;
  }
};

// 认证方式选项
const authTypes = ['密码', '公钥'];
const activeAuthType = ref('密码');

// 处理连接
const handleConnect = () => {
  emit('save', formData.value);
};

// 处理保存
const handleSave = () => {
  emit('save', formData.value);
};

// 处理取消
const handleCancel = () => {
  emit('cancel');
};
</script>

<template>
  <div class="create-connect">
    <div class="content-area">
      <!-- 左侧表单 -->
      <div class="left-form">
        <div class="form-item">
          <div class="label">名称</div>
          <input type="text" v-model="formData.name" placeholder="终端连接" class="input-field" />
        </div>
        
        <div class="form-item">
          <div class="label">分组</div>
          <div class="group-selector">
            <select v-model="formData.group" class="input-field">
              <option value="默认">默认</option>
            </select>
            <button class="add-btn">
              <span>+</span>
            </button>
          </div>
        </div>
        
        <div class="form-item">
          <div class="label">备注</div>
          <textarea v-model="formData.remark" class="textarea-field"></textarea>
        </div>
      </div>
      
      <!-- 右侧表单 -->
      <div class="right-form">
        <!-- 连接类型选项卡 -->
        <div class="connection-tabs">
          <div 
            v-for="type in connectionTypes" 
            :key="type"
            :class="['tab-item', { active: activeType === type }]"
            @click="switchType(type)"
          >
            {{ type }}
          </div>
        </div>
        
        <!-- SSH 表单 -->
        <div v-if="activeType === 'ssh'" class="connection-form">
          <div class="form-item host-port-row">
            <div class="host-field">
              <div class="label">主机</div>
              <input type="text" v-model="formData.host" class="input-field" />
            </div>
            <div class="port-field">
              <div class="label">端口</div>
              <input type="text" v-model="formData.port" class="input-field port-input" />
            </div>
          </div>
          
          <div class="form-item">
            <div class="label">用户名</div>
            <input type="text" v-model="formData.username" class="input-field" />
          </div>
          
          <div class="form-item auth-tabs">
            <div 
              v-for="type in authTypes" 
              :key="type"
              :class="['auth-tab', { active: activeAuthType === type }]"
              @click="activeAuthType = type"
            >
              {{ type }}
            </div>
            <div class="tab-spacer"></div>
            <div class="auth-tab">Keyboard Interactive</div>
          </div>
          
          <div v-if="activeAuthType === '密码'" class="form-item password-item">
            <div class="label">密码</div>
            <div class="password-field">
              <input type="password" v-model="formData.password" class="input-field" />
              <span class="eye-icon">
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#ccc" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"></path>
                  <circle cx="12" cy="12" r="3"></circle>
                </svg>
              </span>
            </div>
          </div>
          
          <div v-if="activeAuthType === '公钥'" class="form-item">
            <div class="label">私钥文件</div>
            <div class="file-input-container">
              <input type="text" class="input-field" placeholder="选择私钥文件" readonly />
              <button class="browse-btn">浏览</button>
            </div>
          </div>
          
          <div class="form-item checkbox-item">
            <input type="checkbox" id="remember-pwd" v-model="formData.rememberPassword" />
            <label for="remember-pwd">记住密码</label>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 底部按钮 -->
    <div class="action-buttons">
      <button class="connect-btn" @click="handleConnect">连接</button>
      <button class="save-btn" @click="handleSave">保存</button>
      <button class="cancel-btn" @click="handleCancel">取消</button>
    </div>
  </div>
</template>

<style scoped>
.create-connect {
  display: flex;
  flex-direction: column;
  height: 100%;
  background-color: #1e2227;
  color: #fff;
  font-family: Arial, sans-serif;
  font-size: 12px;
  position: relative;
}

.content-area {
  display: flex;
  height: calc(100% - 50px); /* 减去底部按钮的高度 */
  overflow-y: auto;
  padding-bottom: 0; /* 减少底部内边距 */
}

.left-form {
  width: 40%;
  padding: 20px;
}

.right-form {
  width: 60%;
  padding: 20px;
  display: flex;
  flex-direction: column;
}

.form-item {
  margin-bottom: 20px;
  position: relative;
}

.password-item {
  margin-bottom: 10px;
}

.label {
  font-size: 12px;
  margin-bottom: 8px;
  color: #ccc;
}

.input-field, .textarea-field {
  width: 100%;
  padding: 10px 12px;
  background-color: #2c3038;
  border: 1px solid #3a3f48;
  border-radius: 4px;
  color: #fff;
  font-size: 12px;
  font-family: Arial, sans-serif;
  box-sizing: border-box;
}

.input-field:focus, .textarea-field:focus {
  border-color: #4a90e2;
  outline: none;
}

.textarea-field {
  min-height: 150px;
  resize: vertical;
}

.group-selector {
  display: flex;
  align-items: center;
}

.add-btn {
  width: 30px;
  height: 36px;
  background-color: #2c3038;
  border: 1px solid #3a3f48;
  border-radius: 4px;
  color: #fff;
  margin-left: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 12px;
}

.add-btn:hover {
  background-color: #3a3f48;
}

.connection-tabs {
  display: flex;
  margin-bottom: 25px;
  border-bottom: 1px solid #3a3f48;
}

.tab-item {
  padding: 10px 15px;
  cursor: pointer;
  text-transform: capitalize;
  font-size: 12px;
  position: relative;
}

.tab-item.active {
  color: #00c853;
}

.tab-item.active::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  width: 100%;
  height: 2px;
  background-color: #00c853;
}

.host-port-row {
  display: flex;
  gap: 15px;
}

.host-field {
  flex: 1;
}

.port-field {
  width: 120px;
}

.port-input {
  width: 100%;
}

.auth-tabs {
  display: flex;
  margin-bottom: 15px;
}

.auth-tab {
  padding: 4px 10px;
  background-color: #2c3038;
  border: 1px solid #3a3f48;
  cursor: pointer;
  margin-right: 0;
  border-radius: 4px;
  font-size: 12px;
  height: 24px;
  line-height: 1;
  display: flex;
  align-items: center;
}

.auth-tab:not(:last-child) {
  margin-right: 5px;
}

.auth-tab.active {
  background-color: #00c853;
  color: #fff;
  border-color: #00c853;
}

.tab-spacer {
  flex: 1;
}

.checkbox-item {
  display: flex;
  align-items: center;
  margin-top: 5px;
  margin-bottom: 0;
  font-size: 12px;
}

.password-field {
  position: relative;
  width: 100%;
}

.eye-icon {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2;
}

.file-input-container {
  display: flex;
  align-items: center;
}

.browse-btn {
  padding: 8px 15px;
  background-color: #2c3038;
  border: 1px solid #3a3f48;
  border-radius: 4px;
  color: #fff;
  margin-left: 10px;
  cursor: pointer;
  font-size: 12px;
  font-family: Arial, sans-serif;
}

.browse-btn:hover {
  background-color: #3a3f48;
}

.action-buttons {
  display: flex;
  justify-content: flex-end;
  padding: 10px 20px;
  background-color: #1e2227;
  height: 50px;
  box-sizing: border-box;
  margin-top: 0; /* 移除顶部边距 */
}

.connect-btn, .save-btn, .cancel-btn {
  padding: 6px 20px;
  border-radius: 4px;
  font-size: 12px;
  cursor: pointer;
  margin-left: 10px;
  min-width: 60px;
  font-weight: 500;
  font-family: Arial, sans-serif;
  height: 28px;
  line-height: 1;
}

.connect-btn {
  background-color: #00c853;
  color: #fff;
  border: none;
}

.connect-btn:hover {
  background-color: #00b34a;
}

.save-btn {
  background-color: #2196f3;
  color: #fff;
  border: none;
}

.save-btn:hover {
  background-color: #1e88e5;
}

.cancel-btn {
  background-color: #f44336;
  color: #fff;
  border: none;
}

.cancel-btn:hover {
  background-color: #e53935;
}

@media (max-width: 768px) {
  .content-area {
    flex-direction: column;
  }
  
  .left-form, .right-form {
    width: 100%;
  }
  
  .host-port-row {
    flex-direction: column;
    gap: 20px;
  }
  
  .port-field {
    width: 100%;
  }
}
</style>