<template>
  <div>
    <b-card style="border-left: 0; border-right:0; border-top:0 ">
      <b-row>
        <b-col sm="5">
          <h4 id="chart-portfolio-vulns" class="card-title mb-0">{{ $t('message.component_vulnerabilities') }}</h4>
          <div class="small text-muted">
            {{$t('message.last_measurement')}}: {{lastMeasurement}}<b-link v-permission="'PORTFOLIO_MANAGEMENT'" class="font-weight-bold" style="margin-left:6px" v-on:click="refreshMetrics"><i class="fa fa-refresh"></i></b-link>
          </div>
        </b-col>
        <b-col sm="7" class="d-none d-md-block">
        </b-col>
      </b-row>
      <chart-portfolio-vulnerabilities ref="chartComponentVulnerabilities" chartId="chartComponentVulnerabilities" class="chart-wrapper" style="height:200px;margin-top:40px;" :height="200"></chart-portfolio-vulnerabilities>
      <div slot="footer">
        <b-row>
          <b-col sm="12" lg="4">
            <b-row>
              <b-col sm="6">
                <CCallout variant="severity-critical">
                  <small class="text-muted">{{ $t('severity.critical') }}</small><br>
                  <strong class="h4">{{currentCritical}}</strong>
                </CCallout>
              </b-col>
              <b-col sm="6">
                <CCallout variant="severity-high">
                  <small class="text-muted">{{ $t('severity.high') }}</small><br>
                  <strong class="h4">{{currentHigh}}</strong>
                </CCallout>
              </b-col>
            </b-row>
          </b-col>
          <b-col sm="12" lg="4">
            <b-row>
              <b-col sm="6">
                <CCallout variant="severity-medium">
                  <small class="text-muted">{{ $t('severity.medium') }}</small><br>
                  <strong class="h4">{{currentMedium}}</strong>
                </CCallout>
              </b-col>
              <b-col sm="6">
                <CCallout variant="severity-low">
                  <small class="text-muted">{{ $t('severity.low') }}</small><br>
                  <strong class="h5">{{currentLow}}</strong>
                </CCallout>
              </b-col>
            </b-row>
          </b-col>
          <b-col sm="12" lg="4">
            <b-row>
              <b-col sm="6">
                <CCallout variant="severity-unassigned">
                  <small class="text-muted">{{ $t('severity.unassigned') }}</small><br>
                  <strong class="h4">{{currentUnassigned}}</strong>
                </CCallout>
              </b-col>
              <b-col sm="6">
                <CCallout variant="severity-info">
                  <small class="text-muted">{{ $t('message.risk_score') }}</small><br>
                  <strong class="h5">{{currentRiskScore}}</strong>
                </CCallout>
              </b-col>
            </b-row>
          </b-col>
        </b-row>
      </div>
    </b-card>
    <b-row>
      <b-col sm="6">
        <b-card>
          <b-row>
            <b-col sm="5">
              <h4 id="chart-policy-violations" class="card-title mb-0">{{ $t('message.policy_violations') }}</h4>
              <div class="small text-muted">
                {{$t('message.policy_violations_by_state')}}
              </div>
            </b-col>
            <b-col sm="7" class="d-none d-md-block">
            </b-col>
          </b-row>
          <chart-policy-violations ref="chartPolicyViolations" chartId="chartPolicyViolations" class="chart-wrapper" style="height:200px;margin-top:40px;" :height="200"></chart-policy-violations>
        </b-card>
      </b-col>
      <b-col sm="6">
        <b-card>
          <b-row>
            <b-col sm="5">
              <h4 id="chart-policy-violation-breakdown" class="card-title mb-0">{{ $t('message.policy_violations') }}</h4>
              <div class="small text-muted">
                {{$t('message.policy_violations_by_classification')}}
              </div>
            </b-col>
            <b-col sm="7" class="d-none d-md-block">
            </b-col>
          </b-row>
          <chart-policy-violation-breakdown ref="chartPolicyViolationBreakdown" chartId="chartPolicyViolationBreakdown" class="chart-wrapper" style="height:200px;margin-top:40px;" :height="200"></chart-policy-violation-breakdown>
        </b-card>
      </b-col>
    </b-row>
  </div>
</template>

<script>
  import ChartPolicyViolationBreakdown from '@/views/dashboard/ChartPolicyViolationBreakdown';
  import ChartPolicyViolations from "@/views/dashboard/ChartPolicyViolations";
  import { CCallout } from '@coreui/vue';
  import common from "../../../shared/common";
  import ChartAuditingProgress from "../../dashboard/ChartAuditingProgress";
  import ChartComponentVulnerabilities from "../../dashboard/ChartComponentVulnerabilities";
  import ChartPortfolioVulnerabilities from '../../dashboard/ChartPortfolioVulnerabilities';

  export default {
    name: 'ComponentDashboard',
    components: {
      ChartAuditingProgress,
      ChartComponentVulnerabilities,
      ChartPortfolioVulnerabilities,
      ChartPolicyViolations,
      ChartPolicyViolationBreakdown,
      CCallout,

    },
    data() {
      return {
        currentCritical: 0,
        currentHigh: 0,
        currentMedium: 0,
        currentLow: 0,
        currentUnassigned: 0,
        currentRiskScore: 0,

        totalComponents: 0,
        vulnerableComponents: 0,
        vulnerableComponentPercent: 0,

        totalFindings: 0,
        auditedFindings: 0,
        auditedFindingPercent: 0,

        vulnerabilities: 0,
        suppressed: 0,
        lastMeasurement: ""
      }
    },
    methods: {
      extractStats(metrics) {
        if (!metrics || metrics.length === 0) {
          return;
        }
        let metric = metrics[metrics.length - 1]; //Use the most recent metric
        this.currentCritical = common.valueWithDefault(metric.critical, 0);
        this.currentHigh = common.valueWithDefault(metric.high, 0);
        this.currentMedium = common.valueWithDefault(metric.medium, 0);
        this.currentLow = common.valueWithDefault(metric.low, 0);
        this.currentUnassigned = common.valueWithDefault(metric.unassigned, 0);
        this.currentRiskScore = common.valueWithDefault(metric.inheritedRiskScore, 0);

        this.totalComponents = common.valueWithDefault(metric.components, "0");
        this.vulnerableComponents = common.valueWithDefault(metric.vulnerableComponents, "0");
        this.vulnerableComponentPercent = common.calcProgressPercent(this.totalComponents, this.vulnerableComponents);

        this.totalFindings = common.valueWithDefault(metric.findingsTotal, "0");
        this.auditedFindings = common.valueWithDefault(metric.findingsAudited, "0");
        this.auditedFindingPercent = common.calcProgressPercent(this.findingsTotal, this.findingsAudited);

        this.vulnerabilities = common.valueWithDefault(metric.vulnerabilities, "0");
        this.suppressed = common.valueWithDefault(metric.suppressed, "0");
        this.lastMeasurement = common.formatTimestamp(metric.lastOccurrence, true);
      },
      refreshMetrics() {
        let uuid = this.$route.params.uuid;
        let url = `${this.$api.BASE_URL}/${this.$api.URL_METRICS}/component/${uuid}/refresh`;
        this.axios.get(url).then((response) => {
          this.$toastr.s(this.$t('message.metric_refresh_requested'));
        });
      }
    },
    mounted() {
      const daysBack = 90;
      let uuid = this.$route.params.uuid;
      let url = `${this.$api.BASE_URL}/${this.$api.URL_METRICS}/component/${uuid}/days/${daysBack}`;
      this.axios.get(url).then((response) => {
        this.$refs.chartComponentVulnerabilities.render(response.data);
        this.$refs.chartPolicyViolations.render(response.data);
        this.$refs.chartPolicyViolationBreakdown.render(response.data);
        this.extractStats(response.data);
      });
    }
  }
</script>
