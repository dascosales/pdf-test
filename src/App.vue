<template>
  <v-app>
    <v-toolbar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>Pdf Test</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn
        flat
        href="https://rawgit.com/MrRio/jsPDF/master/docs/"
        target="_blank"
      >
        <span class="mr-2">jsPDF</span>
      </v-btn>
    </v-toolbar>

    <v-content>
      <router-view />
    </v-content>
  </v-app>
</template>

<script>
export default {
  components: {
    
  },
  data () {
    return {
      //
    }
  }
}
</script>
