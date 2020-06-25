<template>
  <div id="app" class="container-fluid">
    <Header/>
    <Main />
  </div>
</template>

<script>
import Main from './components/Main.vue';
import Header from './components/Header.vue';

export default {
  name: 'App',
  components: {
    Main,
    Header
  }
}
</script>

<style>
html {
  background-color: rgb(33, 87, 45);
  color:azure;
}
</style>
