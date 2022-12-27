<!--
 * @Description:
 * @Author: 华松林
 * @Date: 2021-09-24 16:56:34
 * @LastEditors: 华松林
 * @LastEditTime: 2021-09-26 10:16:02
 * @FilePath: \single-reader\src\renderer\App.vue
-->
<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: 'single-reader'
  }
</script>

<style>
  /* CSS */
  html, body, #app {
    height: 100%;
  }
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }
  body {
    font-family: 'Source Sans Pro', sans-serif;
  }
  /**滚动条的宽度*/
  ::-webkit-scrollbar {
    width: 5px;
    height: 5px;
  }

  /*滚动条的滑块*/
  ::-webkit-scrollbar-thumb {
    background-color: rgba(0,0,0,.2);
    border-radius: 5px;
  }
</style>
