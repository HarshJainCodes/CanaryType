<template>
  
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <MainHeading></MainHeading>
  <TypingArea></TypingArea>
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import MainHeading from './components/MainHeading.vue'
import TypingArea from './components/TypingArea.vue';

export default {
  name: 'App',
  components: {
    // HelloWorld,
    MainHeading,
    TypingArea
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
  margin-top: 60px;
}
</style>
