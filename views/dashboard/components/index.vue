<template>
  <div>
    <div v-if="isEmpty">
      <a-empty style="padding-top: 200px;">
        <a-button type="primary" @click="handleCreateDashboard">
          {{ $t('monitor.dashboard.dialog.create') }}
        </a-button>
      </a-empty>
    </div>
    <div v-else>
      <a-row :gutter="8">
        <a-col :span="8">
          <base-select
              filterable
              resource="alertdashboards"
              optionLabelProp="label"
              v-model="dashboardId"
              :options="dashboards" />
        </a-col>
        <a-col :span="4">
          <a-button type="primary" @click="handleCreateDashboard">
            {{ $t('monitor.dashboard.dialog.create') }}
          </a-button>
        </a-col>
        <a-col :span="11" />
        <a-col :span="1">
          <a-dropdown style="float: right" :trigger="['click']" placement="bottomRight">
            <a class="ant-dropdown-link font-weight-bold pl-2 pr-2 h-100 d-block action-btn" @click="e => e.preventDefault()">
              <icon type="more" style="font-size: 18px;" />
            </a>
            <a-menu slot="overlay" @click="handleActionClick">
<!--              <a-menu-item key="handleCopy"><a-icon type="copy" />{{$t('dashboard.text_107')}}</a-menu-item>-->
              <a-menu-item key="handleDelete"><a-icon type="delete" />{{$t('scope.text_18')}}</a-menu-item>
            </a-menu>
          </a-dropdown>
        </a-col>
      </a-row>
      <a-row>
        <a-divider />
      </a-row>
      <a-row v-if="dashboardId">
        <dashboard-cards :id="dashboardId" :create-chart="createChart"  :edit-chart="editChart" />
      </a-row>
    </div>
  </div>
</template>

<script>
import DialogMixin from '@/mixins/dialog'
import WindowsMixin from '@/mixins/windows'
import DashboardCards from '@Monitor/components/MonitorCard/DashboardCards'
import { getNameDescriptionTableColumn, getProjectTableColumn, getProjectDomainTableColumn } from '@/utils/common/tableColumn'

export default {
  name: 'DashboardIndex',
  components: {
    DashboardCards,
  },
  mixins: [DialogMixin, WindowsMixin],
  data () {
    return {
      scope: this.$store.getters.scope,
      loading: true,
      manager: new this.$Manager('alertdashboards', 'v1'),
      dashboards: [],
      dashboardId: '',
    }
  },
  computed: {
    isEmpty () {
      return !this.dashboards || this.dashboards.length === 0
    },
  },
  created () {
    this.fetchDashboards()
  },
  methods: {
    handleCreateDashboard () {
      this.createDialog('CreateMonitorDashboard', {
        refresh: this.fetchDashboards,
      })
    },
    handleActionClick ({ key }) {
      if (this[key]) this[key]()
    },
    handleDelete () {
      this.createDialog('DeleteMonitorDashboard', {
        data: this.dashboards.filter((item) => { return item.id === this.dashboardId }),
        refresh: this.fetchDashboards,
        columns: [getNameDescriptionTableColumn(), getProjectDomainTableColumn(), getProjectTableColumn()],
      })
    },
    createChart () {
      this.$router.push({
        path: '/monitor-dashboard/create',
        query: {
          dashboard: this.dashboardId,
        },
      })
    },
    editChart ({ id }) {
      if (!id) return
      this.$router.push({
        path: `/monitor-dashboard/${id}/update`,
        query: {
          dashboard: this.dashboardId,
        },
      })
    },
    async fetchDashboards () {
      this.loading = true
      this.dashboards = []
      try {
        const params = {
          scope: this.scope,
        }
        const { data: { data } } = await this.manager.list({ params })
        this.dashboards = data
        if (data && data.length > 0) {
          this.dashboardId = data[0].id
        }
        this.loading = false
      } catch (error) {
        throw error
      } finally {
        this.loading = false
      }
    },
  },
}
</script>