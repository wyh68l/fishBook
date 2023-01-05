<!--
import { MenuItem } from 'electron';
import { log } from 'util';
 * @Description:
 * @Author: 华松林
 * @Date: 2021-09-24 17:05:35
 * @LastEditors: 华松林
 * @LastEditTime: 2021-09-27 11:11:05
 * @FilePath: \single-reader\src\renderer\pages\index.vue
-->
<template>
  <div id="page" :class="{active: mouseInView && isShow}"  @dblclick="showText" @mouseover.prevent="mouseIn(true)" @mouseout.prevent="mouseIn(false)" @contextmenu.prevent="rightClick">
    <div class="header"></div>
    <!-- 如果鼠标不再阅读器上则显示随便放一个网站域名 -->
    <textarea class="preview" :class="(!isShow) && 'hide'" :value="nowRead" readonly @click="changePage">
      <!--{{ mouseInView ? nowRead : 'fanyi.baidu.com/translate?aldtype=16047&query=&keyfrom=baidu&smartresult=dict&lang=auto2zh#en/zh/single-reader' }}-->

      <!--{{(mouseInView && isShow) ? nowRead : ''}}-->
    </textarea>
    <div> {{ book && nowRead.length < lineSize ? '（已读完）' : '' }}</div>
    <!-- {{ keyword }} -->
  </div>
</template>

<script>
  import { ipcRenderer, app } from "electron";
  const { remote } = require('electron')
  const mainWindow = remote.getCurrentWindow()
  export default {
    name: 'index-page',
    data: () => {
      return {
        content: ``,
        lineSize: 1000, // 一行显示38个字
        mouseInView: false,
        menu: null,
        keyword: '',
        isShow:false,
        isLoading:true,
        isLoading2:true,
	      scrollHeight:0
      }
    },
    mounted() {
	    const textarea = document.querySelector(".preview")
	    textarea.addEventListener('scroll',this.Scrollbottom)
    },
    created() {
      // 监听键盘事件
      document.onkeydown = (e) => {
        this.keyupEvent(e)
      }
      this.initMenu()
	    // window.addEventListener('scroll', this.Scrollbottom);  //页面加载时监听滚动事件
    },
    computed: {
      book() {
        return this.$store.state.Book.book
      },
      // 正在阅读的内容
      nowRead() {
        return (this.book && this.book.context) ? this.book.context.slice(this.lineSize * (this.book.page - 1), this.lineSize * this.book.page) : '请先右键选择'
      }
    },
    methods: {
      // 鼠标是否放置在阅读器中
      mouseIn(value) {
        this.mouseInView = value
        if(!value){
        	this.isShow = false
	        mainWindow.setOpacity(0.04)
	        document.body.style.background = '#000'
        }
      },

// methods中
	    Scrollbottom(e) {
		    let scrollTop = e.target.scrollTop;
		    let clientHeight = e.target.clientHeight;
		    let scrollHeight = e.target.scrollHeight;
		    if (scrollTop + clientHeight >= scrollHeight) {
			    if(this.isLoading && this.isShow){
				    // console.log("滚动到底部了")
				    this.isLoading = false
				    // document.documentElement.scrollTop = 10;
				    // document.body.scrollTop = 10;
			    }
		    }
		    if(scrollTop <= 0 && this.isShow){
			    if(this.isLoading2){
				    this.isLoading2 = false
            // console.log(222)
				    this.scrollHeight = scrollHeight
			    }
        }
	    },
      //双击内容区域展示
	    showText(){
      	this.isShow = true
		    mainWindow.setOpacity(.8)
        document.body.style.backgroundColor = '#333'
      },
      // 按键
      keyupEvent(e) {
      	console.log(window.event.keyCode)
        if (!this.mouseInView) return

        switch (window.event.keyCode) {
          case 37:
            // 上一页
            this.prevPage()
            break;
          case 39:
            // 下一页
            this.nextPage()
            break;
          case 65:
            // 上一页
            this.prevPage()
            break;
          case 68:
            // 下一页
            this.nextPage()
            break;
        }
      },
      prevPage() {
        this.$store.dispatch('prevPage')
        const textarea = document.querySelector(".preview")
        textarea.scroll({ top: 10, left: 0, behavior: "smooth" })
	      setTimeout(()=>{
		      this.isLoading2 = true
	      },1000)
      },
      nextPage() {
        // 如果当前正在阅读的内容小于一行显示的文本数量，则说明是最后一行
        if (this.nowRead.length === this.lineSize) {
	        this.$store.dispatch('nextPage')
	        const textarea = document.querySelector(".preview")
	        textarea.scroll({ top: 10, left: 0, behavior: "smooth" })
          setTimeout(()=>{
	          this.isLoading = true
          },1000)
        }
      },
      //滑动到顶部或底部的时候点击左键可翻页
	    changePage() {
		    if (this.nowRead.length === this.lineSize && !this.isLoading) {
			    this.$store.dispatch('nextPage')
			    const textarea = document.querySelector(".preview")
			    textarea.scroll({ top: 10, left: 0, behavior: "smooth" })
			    setTimeout(()=>{
				    this.isLoading = true
				    this.isLoading2 = true
			    },1000)
          return false
		    }

		    if (!this.isLoading2) {
			    this.$store.dispatch('prevPage')
			    const textarea = document.querySelector(".preview")
			    textarea.scroll({ top: this.scrollHeight - 10, left: 0, behavior: "smooth" })
			    setTimeout(()=>{
				    this.isLoading = true
				    this.isLoading2 = true
			    },1000)
		    }
      },
      // 初始化菜单
      initMenu() {
        const Menu = remote.Menu
        const MenuItem = remote.MenuItem
        this.menu = new Menu()
        this.menu.append(new MenuItem({
          label: '菜单',
          click: () => {
            ipcRenderer.send('openMenuWindow')
          }
        }))
      },
      // 右键打开菜单
      rightClick() {
        this.menu.popup(remote.getCurrentWindow())
      }
    }
  }
</script>

<style>
  #page {
    height: 100%;
    display: flex;
    align-items: center;
    /*-webkit-app-region: drag;*/
    /*-webkit-app-region: no-drag;*/
  }
  .active{
    color: #fff;
    /*background: rgba(0,0,0,.8);*/
  }
  .header{
    width: 100%;
    height: 20px;
    background:none;
    position: absolute;
    z-index: 999;
    top: 0;
    left: 0;
    -webkit-app-region: drag;
  }
  .preview {
    width: 100%;
    height: 90%;
    margin: 20px auto;
    margin-top: 0;
    padding: 20px;
    line-height: 23px;
    font-size: 14px;
    background: none;
    border: none;
    color: #fff;
    resize:none;
    outline: none;
    /*background: rgba(0,0,0,.8);*/
    /*background-color: #fff;*/
    /*-webkit-app-region: drag;*/
    /*-webkit-app-region: no-drag;*/

  }
  .hide{
    color: transparent;
  }
</style>
