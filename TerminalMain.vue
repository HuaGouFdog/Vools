<template>
  <div class="terminal-container">
    <div :id="terminalId" class="terminal-instance"></div>
  </div>
</template>

<script>
// 使用全局变量跟踪脚本加载状态
const scriptsLoaded = {
  css: false,
  xterm: false,
  fitAddon: false
};

export default {
  name: 'TerminalView',
  props: {
    // 允许父组件传入唯一标识符
    uniqueId: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      // 为每个终端实例生成唯一ID
      terminalId: 'terminal-' + (this.uniqueId || Math.random().toString(36).substring(2, 15)),
      term: null,
      fitAddon: null,
      socket: null,
      resizeHandler: null,
      terminalResizeListener: null,
      isInitialized: false
    }
  },
  created() {
    // 在created钩子中初始化terminalId，确保唯一性
    this.terminalId = 'terminal-' + (this.uniqueId || Math.random().toString(36).substring(2, 15));
    console.log(`创建终端实例: ${this.terminalId}`);
  },
  mounted() {
    // 使用延迟初始化，确保DOM已完全渲染
    setTimeout(() => {
      // 确保xterm和xterm-addon-fit已加载
      this.loadScripts().then(() => {
        this.initTerminal();
        // 添加一个短暂延迟，确保终端完全初始化
        setTimeout(() => {
          if (this.term && this.fitAddon) {
            try {
              this.fitAddon.fit();
            } catch (e) {
              console.error('终端适配失败:', e);
            }
          }
          window.dispatchEvent(new Event('terminal-resize'));
        }, 300);
      }).catch(error => {
        console.error('加载脚本失败:', error);
      });
    }, 100);
  },
  methods: {
    // 加载必要的脚本
    loadScripts() {
      return new Promise((resolve, reject) => {
        // 确定基础路径
        const baseUrl = import.meta.env.MODE === 'development'
          ? '../../public/lib/xterm/'
          : './lib/xterm/';

        // 如果脚本已加载，直接返回
        if (scriptsLoaded.css && scriptsLoaded.xterm && scriptsLoaded.fitAddon) {
          console.log('脚本已加载，跳过加载过程');
          resolve();
          return;
        }

        const promises = [];

        // 加载xterm CSS
        if (!scriptsLoaded.css) {
          const cssPromise = new Promise((cssResolve, cssReject) => {
            const cssLink = document.createElement('link');
            cssLink.rel = 'stylesheet';
            cssLink.href = `${baseUrl}xterm.css`;
            cssLink.onload = () => {
              scriptsLoaded.css = true;
              cssResolve();
            };
            cssLink.onerror = () => cssReject(new Error('加载xterm CSS失败'));
            document.head.appendChild(cssLink);
          });
          promises.push(cssPromise);
        }

        // 加载xterm JS
        if (!scriptsLoaded.xterm) {
          const xtermPromise = new Promise((xtermResolve, xtermReject) => {
            const xtermScript = document.createElement('script');
            xtermScript.src = `${baseUrl}xterm.js`;
            xtermScript.onload = () => {
              scriptsLoaded.xterm = true;
              xtermResolve();
            };
            xtermScript.onerror = () => xtermReject(new Error('加载xterm失败'));
            document.head.appendChild(xtermScript);
          });
          promises.push(xtermPromise);
        }

        // 加载xterm-addon-fit JS
        if (!scriptsLoaded.fitAddon) {
          const fitAddonPromise = new Promise((fitResolve, fitReject) => {
            const fitAddonScript = document.createElement('script');
            fitAddonScript.src = `${baseUrl}xterm-addon-fit.min.js`;
            fitAddonScript.onload = () => {
              scriptsLoaded.fitAddon = true;
              fitResolve();
            };
            fitAddonScript.onerror = () => fitReject(new Error('加载xterm-addon-fit失败'));
            document.head.appendChild(fitAddonScript);
          });
          promises.push(fitAddonPromise);
        }

        // 等待所有脚本加载完成
        Promise.all(promises)
          .then(() => {
            console.log('所有脚本加载完成');
            resolve();
          })
          .catch(error => {
            console.error('脚本加载失败:', error);
            reject(error);
          });
      });
    },

    // 处理命令
    processCommand(term, command) {
      const cmd = command.trim().toLowerCase();
      const parts = cmd.split(' ');

      switch (parts[0]) {
        case 'help':
          term.write('\r\n可用命令:');
          term.write('\r\n  help - 显示帮助信息');
          term.write('\r\n  echo [文本] - 显示文本');
          term.write('\r\n  clear - 清屏');
          term.write('\r\n  date - 显示当前日期和时间');
          term.write('\r\n  whoami - 显示当前用户');
          term.write('\r\n  id - 显示终端ID');
          term.write('\r\n  exit - 退出终端');
          break;
        case 'echo':
          term.write('\r\n' + parts.slice(1).join(' '));
          break;
        case 'clear':
          term.clear();
          break;
        case 'date':
          term.write('\r\n' + new Date().toString());
          break;
        case 'whoami':
          term.write('\r\n当前用户: admin');
          break;
        case 'id':
          term.write('\r\n终端ID: ' + this.terminalId);
          break;
        case 'exit':
          term.write('\r\n退出终端...');
          setTimeout(() => {
            term.clear();
            term.write('终端已关闭。按Enter键重新启动...');
          }, 500);
          break;
        default:
          if (cmd) {
            term.write(`\r\n命令未找到: ${cmd}`);
          }
      }
    },

    // 初始化终端
    initTerminal() {
      // 检查是否已经初始化
      if (this.isInitialized) {
        console.warn('终端已初始化，跳过重复初始化');
        return;
      }

      // 确保Terminal和FitAddon已定义
      if (typeof Terminal === 'undefined' || typeof FitAddon === 'undefined') {
        console.error('Terminal或FitAddon未定义，请确保脚本已正确加载');

        // 添加重试机制
        setTimeout(() => {
          console.log('尝试重新初始化终端...');
          this.initTerminal();
        }, 500);

        return;
      }

      try {
        const term = new Terminal({
          convertEol: true, // 启用时，光标将设置为下一行的开头
          scrollback: 5, // 终端中的回滚量
          cursorBlink: true, // 光标闪烁
          letterSpacing: 0,
          fontSize: 15,
          fontFamily: 'Cascadia Mono',
          theme: {
            foreground: '#f3f3f3', // 字体
            background: '#2d2d2d', // 背景色
          }
        });

        const fitAddon = new FitAddon.FitAddon();
        term.loadAddon(fitAddon);

        // 使用唯一的终端ID获取元素
        const terminalElement = document.getElementById(this.terminalId);
        if (!terminalElement) {
          console.error(`找不到终端元素: ${this.terminalId}`);
          return;
        }

        term.open(terminalElement);

        // 确保终端适配到容器大小
        try {
          fitAddon.fit();
        } catch (e) {
          console.error('终端适配失败:', e);
        }

        term.write('连接终端中...\r\n');

        // 尝试建立WebSocket连接，使用唯一的会话ID
        let socket;
        try {
          // 为每个终端实例创建唯一的WebSocket连接
          const sessionId = this.terminalId;
          socket = new WebSocket(`ws://localhost:8080/ws/terminal?sessionId=${sessionId}`);

          socket.onopen = () => {
            term.write(`✅ 会话 ${sessionId} 已连接\r\n`);
          };

          socket.onmessage = (e) => {
            term.write(e.data);
          };

          socket.onclose = () => {
            term.write('\r\n❌ 连接关闭\r\n');
          };

          socket.onerror = (error) => {
            console.error('WebSocket错误:', error);
            term.write('\r\n❌ 连接错误\r\n');
          };

          term.onData(data => {
            if (socket && socket.readyState === WebSocket.OPEN) {
              socket.send(data);
            }
          });
        } catch (e) {
          console.error('WebSocket连接失败:', e);
          term.write('\r\n❌ 无法建立连接\r\n');
        }

        // 窗口大小变化时重新 fit
        const resizeHandler = () => {
          try {
            if (fitAddon) fitAddon.fit();
          } catch (e) {
            console.error('终端重新适配失败:', e);
          }
        };

        window.addEventListener('resize', resizeHandler);

        // 初始化后多次尝试适配，确保终端正确显示
        fitAddon.fit();

        // 使用多个延迟时间尝试适配，增加成功率
        const fitDelays = [100, 300, 500, 1000];
        fitDelays.forEach(delay => {
          setTimeout(() => {
            try {
              if (fitAddon) fitAddon.fit();
            } catch (e) {
              console.error(`${delay}ms后终端适配失败:`, e);
            }
          }, delay);
        });

        // 创建终端重新适配的事件监听器
        const terminalResizeListener = () => {
          try {
            if (fitAddon) fitAddon.fit();
          } catch (e) {
            console.error('终端重新适配失败:', e);
          }
        };

        // 监听自定义的终端重新适配事件
        window.addEventListener('terminal-resize', terminalResizeListener);

        // 保存引用以便在组件销毁时清理
        this.term = term;
        this.fitAddon = fitAddon;
        this.socket = socket;
        this.resizeHandler = resizeHandler;
        this.terminalResizeListener = terminalResizeListener;
        this.isInitialized = true;

        // 通知父组件终端已初始化
        this.$emit('terminal-ready');

        console.log(`终端 ${this.terminalId} 初始化完成`);
      } catch (error) {
        console.error('初始化终端时发生错误:', error);
      }
    }
  },
  beforeUnmount() {
    console.log(`销毁终端实例: ${this.terminalId}`);

    // 清理资源
    if (this.socket) {
      this.socket.close();
      this.socket = null;
    }

    if (this.term) {
      this.term.dispose();
      this.term = null;
    }

    if (this.resizeHandler) {
      window.removeEventListener('resize', this.resizeHandler);
      this.resizeHandler = null;
    }

    if (this.terminalResizeListener) {
      window.removeEventListener('terminal-resize', this.terminalResizeListener);
      this.terminalResizeListener = null;
    }

    this.isInitialized = false;
  }
}
</script>

<style>
.demo-tabs > .el-tabs__content {
  padding: 32px;
  color: #6b778c;
  font-size: 32px;
  font-weight: 600;
}

html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  background: #2d2d2d;
}

.terminal-container {
  width: 100%;
  height: 100%;
  text-align: left;
  display: flex;
  flex-direction: column;
  padding: 8px;
  box-sizing: border-box;
  background-color: #2d2d2d;
}

.terminal-instance {
  width: 100%;
  height: 100%;
  flex: 1;
  border-radius: 4px;
  overflow: hidden;
}

/* 美化xterm原生滚动条 */
:deep(.xterm-viewport::-webkit-scrollbar) {
  width: 8px;
  height: 8px;
}

:deep(.xterm-viewport::-webkit-scrollbar-track) {
  background: #2a2a2a;
  border-radius: 4px;
}

:deep(.xterm-viewport::-webkit-scrollbar-thumb) {
  background-color: #555;
  border-radius: 4px;
  transition: all 0.3s ease;
}

:deep(.xterm-viewport::-webkit-scrollbar-thumb:hover) {
  background-color: #666;
}

:deep(.xterm-viewport::-webkit-scrollbar-corner) {
  background: #2a2a2a;
}
</style>