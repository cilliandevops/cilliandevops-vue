<template>
  <div id="terminal" ref="terminal"></div>
</template>
<script>
import { Terminal } from "xterm"
import { FitAddon } from 'xterm-addon-fit'
import "xterm/css/xterm.css"
export default {
  data() {
    return {
      term: "", // 保存terminal实例
      rows: 40,
      cols: 100
    }
  },
  mounted() {
    this.initXterm()
  },
  methods: {
    initXterm() {
      let _this = this
      let term = new Terminal({
        rendererType: "canvas", //渲染类型
        rows: _this.rows, //行数
        cols: _this.cols, // 不指定行数，自动回车后光标从下一行开始
        convertEol: true, //启用时，光标将设置为下一行的开头
        // scrollback: 50, //终端中的回滚量
        disableStdin: false, //是否应禁用输入
        // cursorStyle: "underline", //光标样式
        cursorBlink: true, //光标闪烁
        theme: {
          foreground: "#ECECEC", //字体
          background: "#000000", //背景色
          cursor: "help", //设置光标
          lineHeight: 20
        }
      })
      // 创建terminal实例
      term.open(this.$refs["terminal"])
      // 换行并输入起始符 $
      term.prompt = _ => {
        term.write("\r\n\x1b[33m$\x1b[0m ")
      }
      term.prompt()
      // canvas背景全屏
      const fitAddon = new FitAddon()
      term.loadAddon(fitAddon)
      fitAddon.fit()

      window.addEventListener("resize", resizeScreen)
      function resizeScreen() {
        try { // 窗口大小改变时，触发xterm的resize方法使自适应
          fitAddon.fit()
        } catch (e) {
          console.log("e", e.message)
        }
      }
      _this.term = term
      _this.runFakeTerminal()
    },
    runFakeTerminal() {
      let _this = this
      let term = _this.term
      if (term._initialized) return
      // 初始化
      term._initialized = true
      term.writeln("Welcome to \x1b[1;32m希里安\x1b[0m.")
      term.writeln('This is Web Terminal of Modb; Good Good Study, Day Day Up.')
      term.prompt()
      // 添加事件监听器，支持输入方法
      term.onKey(e => {
        const printable = !e.domEvent.altKey && !e.domEvent.altGraphKey && !e.domEvent.ctrlKey && !e.domEvent.metaKey
        if (e.domEvent.keyCode === 13) {
          term.prompt()
        } else if (e.domEvent.keyCode === 8) { // back 删除的情况
          if (term._core.buffer.x > 2) {
            term.write('\b \b')
          }
        } else if (printable) {
          term.write(e.key)
        }
        console.log(1,'print', e.key)
      })
      term.onData(key => {  // 粘贴的情况
        if(key.length > 1) term.write(key)
      })
    }
  }
}
</script>



<template>
  <div >
    <div ref="terminal" />
  </div>
</template>
<script>
import 'xterm/css/xterm.css'
import { Terminal } from 'xterm'
import { FitAddon } from 'xterm-addon-fit'
import { AttachAddon } from 'xterm-addon-attach'

export default {
  name: 'terminal',
  data() {
    return {
      term: null,
      socketUri: 'ws://127.0.0.1:8088/podname',
      socket: '',
      accessToken: 'token',
    }
  },
  mounted() {
    this.initTerm();
  },
  beforeDestroy() {
    this.socket && this.socket.close();
    this.term && this.term.dispose();
  },
  methods: {
    initTerm() {

      // 1.xterm终端初始化
      const term = new Terminal({
        rendererType: "canvas", //渲染类型
        rows: 40, //行数
        cols: 100, // 不指定行数，自动回车后光标从下一行开始
        convertEol: true, //启用时，光标将设置为下一行的开头
        // scrollback: 50, //终端中的回滚量
        disableStdin: false, //是否应禁用输入
        windowsMode: true, // 根据窗口换行
        cursorStyle: "underline", //光标样式
        cursorBlink: true, //光标闪烁
        theme: {
          foreground: "#ECECEC", //字体
          background: "#000000", //背景色
          cursor: "help", //设置光标
          lineHeight: 20,
        },
      });
      // 2.webSocket初始化
      if (this.socketUri === '') return;
      this.socket = new WebSocket(this.socketUri, this.accessToken);    // 带 token 发起连接
      // 3.websocket集成的插件,这里要注意,网上写了很多websocket相关代码.xterm4版本没必要.
      const attachAddon = new AttachAddon(this.socket);
      const fitAddon = new FitAddon() // 全屏插件
      term.loadAddon(attachAddon);
      term.loadAddon(fitAddon);
      term.open(this.$refs.terminal);
      fitAddon.fit();
      term.focus();
      this.term = term;
    }
  }
}
</script>
