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
:root {
  --app-width: 1200px;
}

.app {
  width: var(--app-width);
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
