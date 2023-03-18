<template>
  <CHeader colorScheme="dark" fixed class="py-2">
    <img v-if="!sidebarShow" style="margin-left: 10px; margin-top: 4px;" src="@/assets/img/brand/dt-logo-symbol.svg" width="30" height="30" alt="Dependency-Track Logo"/>
    <CToggler
      in-header
      class="ml-3 d-lg-none"
      @click="$store.commit('toggleSidebarMobile')"
    />
    <CToggler
      in-header
      class="ml-3 d-md-down-none"
      @click="$store.commit('toggleSidebarDesktop')"
    />
    <b-navbar-nav class="ml-auto">
      <DefaultHeaderProfileDropdown/>
    </b-navbar-nav>
  </CHeader>
</template>
<script>
import { CHeader, CToggler } from '@coreui/vue';
import DefaultHeaderProfileDropdown from './DefaultHeaderProfileDropdown';

export default {
  name: 'DefaultHeader',
  components: {
    CHeader,
    DefaultHeaderProfileDropdown,
    CToggler
  },
  computed: {
    sidebarShow () {
      return this.$store.state.sidebarShow;
    },
  }
}
</script>
