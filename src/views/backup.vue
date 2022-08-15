<template>

</template>

<script>
export default {
  name: "backup"
}
</script>

<style scoped>

</style>

<script>
import 'xterm/css/xterm.css'
import 'xterm/lib/xterm.js'
import { Terminal } from 'xterm'
import { FitAddon } from 'xterm-addon-fit'
import { AttachAddon } from 'xterm-addon-attach'

export default {
name: 'Xterm',
props: {
socketURI: {
  type: String,
  default: ''
},
},
watch: {
socketURI: {
deep: true, //对象内部属性的监听，关键。
              immediate: true,
handler() {
  this.initSocket();
},
},
},
mounted() {
  this.initSocket()
  window.addEventListener('resize', this.onTerminalResize);
},
beforeDestroy() {
  this.socket.close()
  this.term&&this.term.dispose()
},
methods: {
initTerm() {
// this.initSocket();
let term = new Terminal({
rendererType: "canvas", //渲染类型
                          rows: this.rows, //行数
                                             cols: this.cols, // 不指定行数，自动回车后光标从下一行开始
                                                                 convertEol: true, //启用时，光标将设置为下一行的开头
// scrollback: 50, //终端中的回滚量
                     disableStdin: false, //是否应禁用输入
                                            windowsMode: true, // 根据窗口换行
// cursorStyle: "underline", //光标样式
                               cursorBlink: true, //光标闪烁
                                                    theme: {
                                                      foreground: "#ECECEC", //字体
                                                      background: "#000000", //背景色
                                                      cursor: "help", //设置光标
                                                      lineHeight: 20,
                                                    },
});
this.term = term;
const fitAddon = new FitAddon();
this.term.loadAddon(fitAddon);
this.fitAddon = fitAddon;
let element = document.getElementById("xterm");
term.open(element);
// 自适应大小(使终端的尺寸和几何尺寸适合于终端容器的尺寸)，初始化的时候宽高都是对的
                               fitAddon.fit();
term.focus();
//监视命令行输入
  this.term.onData((data) => {
                             let dataWrapper = data;
if (dataWrapper === "\r") {
  dataWrapper = "\n";
} else if (dataWrapper === "\u0003") {
// 输入ctrl+c
dataWrapper += "\n";
}
// 将输入的命令通知给后台，后台返回数据。
   this.socket.send(JSON.stringify({ type: "input", data: dataWrapper }));
});
window.addEventListener("resize", this.onTerminalResize);
},
onTerminalResize() {
  const terminalContainer = document.getElementById("xterm");
  const width = terminalContainer.parentElement.clientWidth;
  const height = terminalContainer.parentElement.clientHeight;
const { term } = this;
// 计算cols，rows
   const cols =
(width - term._core.viewport.scrollBarWidth - 15) /
term._core._renderService._renderer.dimensions.actualCellWidth;
const rows =
height /
term._core._renderService._renderer.dimensions.actualCellHeight -
1;
this.term.resize(
parseInt(cols.toString(), 10),
parseInt(rows.toString(), 10)
);
},
initSocket() {
if (this.socketURI == "") {
  return;
}
this.socket = new WebSocket(this.socketURI);
this.socketOnClose();
this.socketOnOpen();
this.socketOnmessage();
this.socketOnError();
},
socketOnOpen() {
this.socket.onopen = () => {
                           console.log("web链接成功");
                         // 链接成功后
                         this.initTerm();
                           this.onTerminalResize();
                         };
},
socketOnmessage() {
this.socket.onmessage = (evt) => {
try {
if (typeof evt.data === "string") {
  const msg = evt.data;
// 将返回的数据写入xterm，回显在webshell上
this.term.write(msg);
// 此处可以判断一下，如果是首次连接，需要将rows，cols传给服务器端
// when server ready for connection,send resize to server
this.socket.send(
JSON.stringify({
  type: "resize",
  rows: this.term.rows,
  cols: this.term.cols,
})
);
}
} catch (e) {
  console.error(e);
  console.log("parse json error.", evt.data);
}
};
},
socketOnClose() {
this.socket.onclose = () => {
                            this.socket.close();
                            console.log("关闭 socket");
// if (this.socket) {
//   this.socket.close();
// }
window.removeEventListener("resize", this.onTerminalResize);
};
},
socketOnError() {
this.socket.onerror = () => {
                            console.log("socket 链接失败");
                          };
},
}
</script>
