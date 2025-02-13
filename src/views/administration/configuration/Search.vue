<template>
  <div>
    <p>{{ $t('admin.index_general_description') }}</p>
    <p>{{ $t('admin.index_use_cases') }}</p>
    <p>{{ $t('admin.index_issues_description') }}</p>
    <br/>
    <b-card no-body :header="headers.consistencyCheck">
      <b-card-body>
        <p>{{ $t('admin.index_consistency_check_description') }}</p>
        <CSwitch id="consistency-check-enabled" color="primary" :checked.sync="consistencyCheck.enabled" label />{{$t('admin.enable_index_consistency_check')}}
        <b-validated-input-group-form-input
          id="consistency-check-cadence"
          :label="$t('admin.index_consistency_check_cadence')"
          input-group-size="mb-3"
          rules="required|min_value:1"
          type="number"
          v-model="consistencyCheck.cadence"
          :tooltip="consistencyCheck.cadenceTooltip"
        />
        <b-validated-input-group-form-input
          id="consistency-check-threshold"
          :label="$t('admin.index_consistency_check_threshold')"
          input-group-size="mb-3"
          rules="required|min_value:1|max_value:100"
          type="number"
          v-model="consistencyCheck.threshold"
          :tooltip="consistencyCheck.thresholdTooltip"
        />
      </b-card-body>
      <b-card-footer>
        <b-button size="md" class="px-4" variant="outline-primary" @click="saveConsistencyCheckSettings">{{ $t('message.update') }}</b-button>
      </b-card-footer>
    </b-card>
    <b-card no-body :header="headers.manualRebuild">
      <b-card-body>
        <p>{{ $t('admin.index_rebuild_description') }}</p>
        <CSwitch id="project" color="primary" :checked.sync="type.project" label />{{$t('admin.reindex_projects')}}
        <br/>
        <CSwitch id="component" color="primary" :checked.sync="type.component" label />{{$t('admin.reindex_components')}}
        <br/>
        <CSwitch id="vulnerability" color="primary" :checked.sync="type.vulnerability" label />{{$t('admin.reindex_vulnerabilities')}}
        <br/>
        <CSwitch id="vulnerablesoftware" color="primary" :checked.sync="type.vulnerablesoftware" label />{{$t('admin.reindex_vulnerable_software')}}
        <br/>
        <CSwitch id="servicecomponent" color="primary" :checked.sync="type.servicecomponent" label />{{$t('admin.reindex_service_components')}}
        <br/>
        <CSwitch id="license" color="primary" :checked.sync="type.license" label />{{$t('admin.reindex_licenses')}}
        <br/>
        <CSwitch id="cpe" color="primary" :checked.sync="type.cpe" label />{{$t('admin.reindex_cpes')}}
        <br/>
      </b-card-body>
      <b-card-footer>
        <b-button size="md" class="px-4" variant="outline-primary" @click="reindex">{{ $t('message.reindex') }}</b-button>
      </b-card-footer>
    </b-card>
  </div>
</template>

<script>
  import { CSwitch } from '@coreui/vue';
  import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput';
  import common from "../../../shared/common";
  import configPropertyMixin from "../mixins/configPropertyMixin";

  export default {
    mixins: [configPropertyMixin],
    props: {
      header: String
    },
    components: {
      CSwitch,
      BValidatedInputGroupFormInput
    },
    data() {
      return {
        headers: {
          consistencyCheck: this.header + " - Periodic consistency check",
          manualRebuild: this.header + " - Manual rebuild"
        },
        type: {
          project: false,
          component: false,
          vulnerability: false,
          vulnerablesofware: false,
          license: false,
          cpe: false,
          servicecomponent:false
        },
        consistencyCheck: {
          enabled: false,
          cadence: "4320",
          cadenceTooltip: "",
          threshold: "20",
          thresholdTooltip: ""
        }
      }
    },
    created () {
      this.axios.get(this.configUrl).then((response) => {
        let configItems = response.data.filter(function (item) { return item.groupName === "search-indexes" });
        for (let i=0; i<configItems.length; i++) {
          let item = configItems[i];
          switch (item.propertyName) {
            case "consistency.check.enabled":
              this.consistencyCheck.enabled = common.toBoolean(item.propertyValue); break;
            case "consistency.check.cadence":
              this.consistencyCheck.cadence = item.propertyValue;
              this.consistencyCheck.cadenceTooltip = item.description; break;
            case "consistency.check.delta.threshold":
              this.consistencyCheck.threshold = item.propertyValue;
              this.consistencyCheck.thresholdTooltip = item.description; break;
          }
        }
      });
    },
    methods: {
      saveConsistencyCheckSettings: function() {
        this.updateConfigProperties([
          {groupName: 'search-indexes', propertyName: 'consistency.check.enabled', propertyValue: this.consistencyCheck.enabled},
          {groupName: 'search-indexes', propertyName: 'consistency.check.cadence', propertyValue: this.consistencyCheck.cadence},
          {groupName: 'search-indexes', propertyName: 'consistency.check.delta.threshold', propertyValue: this.consistencyCheck.threshold}
        ]);
      },
      reindex: function() {
        let url = `${this.$api.BASE_URL}/${this.$api.URL_SEARCH}/reindex`;
        let params = new URLSearchParams();
        Object.entries(this.type).forEach(([key, value]) => {
          if(value) {
            params.append('type', key.toUpperCase());
          }
        });
        this.axios.post(url, null, { params: params }).then((response) => {
          this.$toastr.s(this.$t('admin.reindex_submitted'));
        }).catch((error) => {
          this.$toastr.s(this.$t('admin.reindex_error'));
        });
      }
    }
  }
</script>
