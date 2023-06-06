<template>
  <div id="app">
    <ChessboardComponent />
  </div>
</template>


<script>


import ChessboardComponent from "@/components/ChessboardComponent";

export default {
  name: 'App',
  components: {
    ChessboardComponent
  }
}
</script>

<style>
#app {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
</style>

