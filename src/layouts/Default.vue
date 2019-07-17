<template lang='pug'>
q-layout(view='lHh Lpr lFf')
  q-header(elevated)
    q-toolbar
      q-toolbar-title Vugify
      q-btn(round flat icon='help')
        q-tooltip Currently lacking multi-line text support and customizability. It's predefined with what I use; will add sooner or later.
  q-page-container
    router-view
</template>

<script>
import { openURL } from 'quasar'

export default {
  name: 'Default',
  data () {
    return {
      leftDrawerOpen: this.$q.platform.is.desktop
    }
  },
  methods: {
    openURL
  }
}
</script>

<style>
</style>
