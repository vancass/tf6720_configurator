<template>
  <div id="app"> 
    <button @click="arburgPage(false)">Normal</button>
    <button @click="arburgPage(true)">Arburg</button> 
    <MainPage v-show="!arburg"></MainPage>
    <Arburg v-show="arburg"></Arburg>
  </div>
</template>

<script>
import MainPage from "./views/MainPage.vue";
import Arburg from "./views/Arburg.vue";

export default {
  name: "app",
  components: {
    MainPage,
    Arburg
  },
  data: function() {
    return {
      arburg: true
    };
  },

  methods: {
    arburgPage(value) {
      this.arburg = value;
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
