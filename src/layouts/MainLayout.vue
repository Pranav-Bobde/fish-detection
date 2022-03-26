<template>
  <q-layout view="lHh Lpr lFf" :style="bgImage">
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>

export default {
  name: 'MainLayout',
  data () {
    return {
      leftDrawerOpen: false,
    }
  },
  computed: {
    bgImage() {
      return {
        backgroundImage: `url(${require('../assets/fish1.jpg')})`
      };
    },
  },
}
</script>
