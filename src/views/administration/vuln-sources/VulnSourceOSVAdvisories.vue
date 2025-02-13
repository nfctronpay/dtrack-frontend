<template>
  <b-card no-body :header="header">
    <b-card-body>
      <img alt="OSV logo" src="@/assets/img/osv-logo.png" width="65"/>
      <hr/>
      <CSwitch
        color="primary"
        id="vulnsourceEnabled"
        label
        :checked.sync="vulnsourceEnabled"
        :disabled="enabledEcosystems.length === 0"
      />
      {{$t('admin.vulnsource_osv_advisories_enable')}}
      <br/>
      <c-switch
        color="primary"
        id="aliasSyncEnabled"
        label
        v-bind="labelIcon"
        v-model="aliasSyncEnabled"
        :title="$t('admin.vulnsource_alias_sync_enable_tooltip')"
        :disabled="!vulnsourceEnabled"
      />
      {{$t('admin.vulnsource_alias_sync_enable')}}
      <p class="font-sm text-muted">
        <span class="fa fa-warning">&nbsp;</span>
        <a :href="aliasGitHubIssueUrl" target="_blank">{{ $t('admin.vulnsource_osv_alias_sync_warning') }}</a>
      </p>
      <hr/>
      {{ $t('admin.vulnsource_osv_advisories_desc') }}
      <hr/>
      <b-validated-input-group-form-input
        id="nvd-feeds-url"
        :label="$t('admin.vulnsource_osv_base_url')"
        input-group-size="mb-3"
        rules="required"
        type="text"
        v-model="osvBaseUrl"
        lazy="true"
      />
      <hr/>
      <b-form-group label="Ecosystems">
        <div class="list-group" style="width: 40%">
          <span v-for="ecosystem in enabledEcosystems" :key="ecosystem">
            <actionable-list-group-item :value="ecosystem" :delete-icon="true" @actionClicked="removeEcosystem(ecosystem)"/>
          </span>
          <actionable-list-group-item :add-icon="true" @actionClicked="$root.$emit('bv::show::modal', 'ecosystemModal')"/>
        </div>
      </b-form-group>
    </b-card-body>
    <b-card-footer>
      <b-button
        :disabled="this.vulnsourceEnabled && !this.osvBaseUrl"
        variant="outline-primary"
        class="px-4"
        @click="saveUrl">
          {{ $t('message.update') }}
      </b-button>
    </b-card-footer>
    <ecosystem-modal v-on:selection="updateEcosystem"/>
  </b-card>
</template>

<script>
import { CSwitch } from '@coreui/vue';
import common from "../../../shared/common";
import configPropertyMixin from "../mixins/configPropertyMixin";
import EcosystemModal from "./EcosystemModal";

export default {
  mixins: [configPropertyMixin],
  props: {
    header: String
  },
  components: {
    CSwitch,
    EcosystemModal
},
  data() {
    return {
      vulnsourceEnabled: false,
      aliasSyncEnabled: false,
      aliasGitHubIssueUrl: "https://github.com/google/osv.dev/issues/888",
      osvBaseUrl: '',
      ecosystemConfig: null,
      enabledEcosystems: [],
    }
  },
  watch: {
    vulnsourceEnabled(toggleChange) {
      if(toggleChange === false) {
        this.enabledEcosystems = [];
        this.updateConfigProperties([
          {groupName: 'vuln-source', propertyName: 'google.osv.enabled', propertyValue: null}
        ]);
      }
    }
  },
  methods: {
    removeEcosystem: function(ecosystem) {
      this.enabledEcosystems = this.enabledEcosystems.filter(e => e !== ecosystem);
      this.vulnsourceEnabled = this.enabledEcosystems.length !== 0;
    },
    updateEcosystem: function(ecosystems) {
      this.$root.$emit('bv::hide::modal', 'ecosystemModal');
      for(let i=0; i<ecosystems.length; i++) {
        let ecosystem = ecosystems[i];
        this.enabledEcosystems.push(ecosystem.name);
      }
      this.vulnsourceEnabled = this.enabledEcosystems.length !== 0;
    },
    saveUrl: function() {
      this.updateConfigProperties([
        {groupName: 'vuln-source', propertyName: 'google.osv.base.url', propertyValue: this.osvBaseUrl},
        {groupName: 'vuln-source', propertyName: 'google.osv.enabled', propertyValue: this.enabledEcosystems.join(";")},
        {groupName: 'vuln-source', propertyName: 'google.osv.alias.sync.enabled', propertyValue: this.aliasSyncEnabled}
      ]);
    }
  },
  created () {
    this.axios.get(this.configUrl).then((response) => {
      let configItems = response.data.filter(function (item) { return item.groupName === "vuln-source" });
      for (let i=0; i<configItems.length; i++) {
        let item = configItems[i];
        switch (item.propertyName) {
          case "google.osv.enabled":
            this.ecosystemConfig = item.propertyValue;
            this.vulnsourceEnabled = this.ecosystemConfig != null;
            break;
          case "google.osv.alias.sync.enabled":
            this.aliasSyncEnabled = common.toBoolean(item.propertyValue);
            break;
          case "google.osv.base.url":
            this.osvBaseUrl = item.propertyValue;
            break;
        }
      }
      this.enabledEcosystems = this.ecosystemConfig.split(';').map(ecosystem => ecosystem.trim());
    });
  }
}
</script>
