<template>
  <div class="app">
    <CSidebar fixed
        :minimize="minimize"
        :show="show"
        @update:show="(value) => $store.commit('set', ['sidebarShow', value])">
        <CSidebarBrand>
          <img v-if="minimize" style="margin-left: 10px;" src="@/assets/img/brand/dt-logo-symbol.svg" width="30" height="30" alt="Dependency-Track Logo"/>
          <img v-if="!minimize" style="margin-left: 1rem;" src="@/assets/img/brand/dt-logo-white-text.svg" width="140" height="30" alt="Dependency-Track Logo"/>
        </CSidebarBrand>
      <CSidebarNav>
        <template v-for="item in permissibleNav">
          <CSidebarNavTitle :key="item.url" v-if="item.title">{{ item.name }}</CSidebarNavTitle>
          <CSidebarNavItem :key="item.url" v-if="!item.title"
            :name="item.name"
            :fontIcon="item.icon"
            :to="item.url"
            :label="item.title"
            :class="item.class"
          />
        </template>
      </CSidebarNav>
      <CSidebarMinimizer
        @click.native="$store.commit('set', ['sidebarMinimize', !minimize])"
      />
    </CSidebar>
    <CWrapper>
      <DefaultHeader/>
      <div class="app-body">
        <main class="main">
          <CBreadcrumb :items="breadcrumbItems"/>
          <div class="container-fluid">
            <router-view></router-view>
          </div>
        </main>
      </div>
      <DefaultFooter/>
    </CWrapper>
    <profile-edit-modal />
    <snapshot-modal/>
  </div>
</template>

<script>
import { CBreadcrumb } from '@coreui/vue';
import { CSidebar, CSidebarBrand, CSidebarMinimizer, CSidebarNav, CSidebarNavItem, CSidebarNavTitle, CWrapper } from '@coreui/vue/src/components/template';
import EventBus from '../shared/eventbus';
import * as permissions from '../shared/permissions';
import ProfileEditModal from "../views/components/ProfileEditModal";
import SnapshotModal from "../views/components/SnapshotModal";
import DefaultFooter from './DefaultFooter';
import DefaultHeader from './DefaultHeader';
import DefaultHeaderProfileDropdown from './DefaultHeaderProfileDropdown';

export default {
  name: 'DefaultContainer',
  components: {
    ProfileEditModal,
    SnapshotModal,
    CWrapper,
    CSidebar,
    CBreadcrumb,
    DefaultHeaderProfileDropdown,
    CSidebarNavTitle,
    CSidebarBrand,
    CSidebarNav,
    CSidebarNavItem,
    CSidebarMinimizer,
    DefaultFooter,
    DefaultHeader
  },
  data () {
    return {
      breadcrumbs: [],
      nav: [
      {
        name: this.$t('message.dashboard'),
        url: '/dashboard',
        icon: 'icon-speedometer',
        permission: permissions.VIEW_PORTFOLIO
      },
      {
        title: true,
        name: this.$t('message.portfolio'),
        class: '',
        wrapper: {
          element: '',
          attributes: {}
        },
        permission: permissions.VIEW_PORTFOLIO
      },
      {
        name: this.$t('message.projects'),
        url: '/projects',
        icon: 'fa fa-sitemap',
        permission: permissions.VIEW_PORTFOLIO
      },
      {
        name: this.$t('message.components'),
        url: '/components',
        icon: 'fa fa-cubes',
        permission: permissions.VIEW_PORTFOLIO
      },
      {
        name: this.$t('message.vulnerabilities'),
        url: '/vulnerabilities',
        icon: 'fa fa-shield',
        permission: permissions.VIEW_PORTFOLIO
      },
      {
        name: this.$t('message.licenses'),
        url: '/licenses',
        icon: 'fa fa-balance-scale',
        permission: permissions.VIEW_PORTFOLIO
      },
      {
        title: true,
        name: this.$t('message.administration'),
        class: '',
        wrapper: {
          element: '',
          attributes: {}
        },
        permission: permissions.SYSTEM_CONFIGURATION
      },
      {
        name: this.$t('message.policy_management'),
        url: '/policy',
        icon: 'fa fa-list-alt',
        permission: permissions.POLICY_MANAGEMENT
      },
      {
        name: this.$t('message.administration'),
        url: '/admin',
        icon: 'fa fa-cogs',
        permission: permissions.SYSTEM_CONFIGURATION
      }
    ]
    }
  },
  methods: {
    generateCBreadcrumbs: function generateCBreadcrumbs(crumbName, subSectionName, subSectionUuid, subSectionLabel) {
      let sectionLabel = this.$t(this.$route.meta.i18n);
      let sectionPath = this.$route.meta.sectionPath;
      if (crumbName && subSectionName && subSectionUuid && subSectionLabel) {
        return [
          { href: '', text: this.$t('message.home') },
          { href: sectionPath, text: sectionLabel },
          { text: subSectionLabel, href: sectionPath + '/' + subSectionUuid },
          { text: crumbName, active: true }
        ];
      } else if (crumbName) {
        return [
          { href: '', text: this.$t('message.home') },
          { href: sectionPath, text: sectionLabel },
          { text: crumbName, active: true }
        ];
      } else {
        return [
          { href: '', text: this.$t('message.home') },
          { href: sectionPath, text: sectionLabel }
        ];
      }
    }
  },
  mounted() {
    if (this.$dtrack && this.$dtrack.version.includes("SNAPSHOT")) {
      this.$root.$emit('bv::show::modal', 'snapshotModal');
    }
  },
  computed: {
    name () {
      return this.$route.name
    },
    breadcrumbItems () {
      if (this.breadcrumbs.length === 0) {
        return this.generateCBreadcrumbs();
      } else {
        return this.breadcrumbs;
      }
    },
    permissibleNav () {
      let decodedToken = permissions.decodeToken(permissions.getToken());
      let array = [];
      for (const item of this.nav) {
        if (item.permission !== null && permissions.hasPermission(item.permission, decodedToken)) {
          array.push(item);
        }
      }
      return array;
    },
    show () {
      return this.$store.state.sidebarShow
    },
    minimize () {
      return this.$store.state.sidebarMinimize
    },
  },
  created() {
    EventBus.$on('crumble', () => {
      this.breadcrumbs = [];
    });
    EventBus.$on('addCrumb', (crumb, subSectionName, subSectionUuid, subsectionLabel) => {
      if (crumb) {
        this.breadcrumbs = this.generateCBreadcrumbs(crumb, subSectionName, subSectionUuid, subsectionLabel);
      } else {
        this.breadcrumbs = [];
      }
    });
  }
}
</script>
