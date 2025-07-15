<template>
  <MainHeader />
  <MainDetail />
  <!-- <DataTable /> -->
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import MainHeader from './components/Header.vue'
import MainDetail from './components/Detail.vue'
// import DataTable from './components/DataTable.vue';

export default {
  name: 'App',
  components: {
    MainHeader,
    MainDetail

  }
}
</script>

<style>
#app {
  margin: 0;
}
</style>
