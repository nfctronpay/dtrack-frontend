<template>
  <footer class="c-footer c-footer-dark">
    <div>
      <!-- left side here -->
    </div>
    <div class="ml-auto">
      <a href="#" v-b-modal.aboutModal >{{ dtrack.application }}</a>&nbsp;
      <span class="mr-1">v{{ dtrack.version }}</span>
    </div>
    <about-modal />
  </footer>
</template>
<script>
  import globalVarsMixin from "../mixins/globalVarsMixin";
  import AboutModal from "../views/components/AboutModal";

  export default {
    name: 'DefaultFooter',
    mixins: [globalVarsMixin],
    components: {
      AboutModal
    },
  }
</script>
<style scoped>
  footer {
    flex: 0 0 24px;
    font-size: 85%;
  }
</style>
