<template>
  <div id="app">
    <PageContentComponent/>
  </div>
</template>

<script>
import PageContentComponent from './components/PageContentComponent.vue'

export default {
  name: 'App',
  components: {
    PageContentComponent
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
body{
  margin: 0;
  font-family: 'Segoe UI', sans-serif;
  background-color: #ECECEC;
}
.navbar{
  height: 70px;
  width: 100%;
  display: flex;
  justify-content: center;
  position: relative;
}
.navbar > h1 {
  font-size: 25px;
  font-weight: bold;
  color: rgb(126, 126, 126);
  z-index: 4;
  margin: 15px 0 5px 0;
}
.top_header_separator {
  position: absolute;
  bottom: 0;
  width: 75%;
  right: 30px;
  margin: -10px auto 10px auto;
  color: #cecdca71;
}
.navbar_left_mask {
  position: absolute;
  width: 12vw;
  min-width: 100px;
  height: 100%;
  background-color: #395477;
  top: 0;
  left: 0;
}
.navbar_left_mask::before {
  display: block;
  content: " ";
  position: absolute;
  height: 140px;
  width: 140px;
  right: -70px;
  top: -70px;
  background: #ececec;
  border-radius: 100%;
}
.layout {
  width: 100%;
  min-height: calc(100vh - 70px);
  display: flex;
  margin: auto;
  flex-wrap: wrap;
}
::placeholder {
  color: rgb(175, 175, 175);
  opacity: 1; 
}
</style>
