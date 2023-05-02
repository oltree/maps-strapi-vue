<template>
  <div class="app">
    <Maps />
  </div>
</template>

<script>
import Maps from './components/Maps.vue';

export default {
  name: 'App',
  components: {
    Maps,
  },
};
</script>

<style>
.app {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
