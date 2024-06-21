<template>
  <div id="app">
    <div class="titleBar">
      <span class="iconfont icon-adjust"></span>
      <div class="rightIcon">
        <span @click="minisize" class="iconfont icon-minus"></span>
        <span v-if="!isMaxSize" @click="screen_zoom" class="iconfont icon-fullscreen-expand"></span>
        <span v-else @click="screen_zoom" class="iconfont icon-fullscreen-shrink"></span>
        <span @click="close" class="iconfont icon-close"></span>
      </div>
    </div>
  </div>
</template>

<script>
import "../public/icon/iconfont.css"
import { ipcRenderer, remote } from "electron";
import debounce from 'lodash/debounce'
export default {
  name: 'App',
  data() {
    return {
      isMaxSize: false
    }
  },
  methods: {
    close() {
      ipcRenderer.send("app", "close")
    },  
    minisize() {
      ipcRenderer.send("app", "minisize")
    },
    screen_zoom() {
      ipcRenderer.send("app", "screen_zoom", this.isMaxSize)
    },
    setState() {
      let win = remote.getCurrentWindow()
      let rect = win.getBounds()
      let isMaxSize = win.isMaximized()
      let obj = { rect, isMaxSize }
      localStorage.setItem("winState", JSON.stringify(obj))
    },
    getState() {
      let win = remote.getCurrentWindow()
      let winState = localStorage.getItem('winState')
      if (winState) {
        winState = JSON.parse(winState)
        if (winState.isMaxSize) {
          win.maximize()
        } else {
          win.setBounds(winState.rect)
        }
      }
    },
    my_debounce: debounce(function() {
      console.log("防抖");
      this.setState();
    },300)
  },
  mounted() {
    let win = remote.getCurrentWindow();
    win.on('maximize', () => {
      this.isMaxSize = true
      this.setState()
    }),
    win.on('unmaximize', () => {
      this.isMaxSize = false
      this.setState()
    })
    win.on('move',() => {
      this.my_debounce()
    })
    win.on("resize",() => {
      this.my_debounce()
    })
    //禁止通过按键刷新
    window.onkeydown = function (e) {
      if (e.keyCode == 82 && (e.ctrlKey || e.metaKey)) return false
    }
    this.isMaxSize = win.isMaximized()
    this.getState()
  }
}
</script>

<style>
html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  overflow: hidden;
  border: 1px solid rgb(190, 194, 194);

}

.titleBar {
  display: flex;
  flex-direction: row;
  align-items: center;
  height: 30px;
  line-height: 30px;
  background-color: #f3f3f3;
  -webkit-app-region: drag;
}

.icon-close,
.icon-adjust,
.icon-fullscreen-expand,
.icon-minus,
.icon-fullscreen-shrink {
  padding: 0 10px;
  height: 100%;
  -webkit-app-region: no-drag;
}

.icon-close:hover {
  background-color: red;
  color: aliceblue;
}

.icon-adjust:hover {
  background-color: rgb(226, 222, 222);
}

.icon-fullscreen-expand:hover {
  background-color: rgb(226, 222, 222);
}

.icon-fullscreen-shrink:hover {
  background-color: rgb(226, 222, 222);
}

.icon-minus:hover {
  background-color: rgb(226, 222, 222);
}

.rightIcon {
  display: flex;
  margin-left: auto;
}
</style>