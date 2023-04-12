<template>
  <div id="analytics">
    <base-header class="pb-6" type="">
      <div class="row align-items-center py-4">
        <div class="col-12 col-sm-4 col-md-6 mb-3 mb-sm-0">
          <h6 class="h2 d-inline-block mb-0">Analytics</h6>
        </div>
        <div class="col-12 col-sm-8 col-md-6">
          <div class="d-flex justify-content-sm-end">
            <div class="d-inline-block mr-4" style="max-width: 300px">
              <label>Filter date range</label>
              <date-range-picker
                ref="picker"
                opens="left"
                :disabled="['Y', 'Q', 'M'].includes(timeframe)"
                :class="['Y', 'Q', 'M'].includes(timeframe) ? 'disabled' : ''"
                :locale-data="{ firstDay: 1, format: 'dd-mm-yyyy' }"
                :time-picker="true"
                :max-date="new Date()"
                customRangeLabel="Custom Range"
                v-model="dateRange"
                @update="reloadAnalytics"
                :ranges="ranges"
              >
                <template v-slot:input="picker" style="min-width: 300px;">
                  {{ picker.startDate | localDate }} - {{ picker.endDate | localDate }}
                </template>
              </date-range-picker>
            </div>

            <div class="d-inline-block" style="max-width: 150px">
              <label>Group by</label>
              <el-select v-model="timeframe" v-on:change="reloadAnalytics(false)"
                         filterable class="w-100">
                <el-option
                  v-for="timeframe in allTimeframes"
                  :key="timeframe.value"
                  :label="timeframe.label"
                  :value="timeframe.value">
                </el-option>
              </el-select>
            </div>
          </div>
        </div>
      </div>
    </base-header>

    <!--Statistics cards-->
    <div class="container-fluid mt--6">
      <div class="row">
        <div class="col-12 col-sm-6 col-lg-3">
          <stats-card title="Total revenue"
                      type="gradient-green"
                      :sub-title="statsCards.totalRevenue | restaurantCurrency"
                      :loading="statsCards.totalRevenue == null"
                      icon="ni ni-money-coins">
            <template slot="footer">
              <span v-if="statsCards.totalRevenueDiff > 0" class="text-success mr-2" v-html="'<i class=\'fa fa-arrow-up\'></i> +' + statsCards.totalRevenueDiff + '%'"></span>
              <span v-if="statsCards.totalRevenueDiff === 0" class="mr-2" v-html="'0%'"></span>
              <span v-if="statsCards.totalRevenueDiff === 'inf'" class="mr-2" v-html="'+ <i class=\'fas fa-infinity text-muted\' style=\'margin-right: 2px;\'></i>%'"></span>
              <span v-if="statsCards.totalRevenueDiff < 0" class="text-danger mr-2" v-html="'<i class=\'fa fa-arrow-down\'></i> ' + statsCards.totalRevenueDiff + '%'"></span>
              <span class="text-nowrap">
                <el-tooltip :content="statsCards.totalRevenueDiff === 'inf' ? statsCards.periodTooltip.replace('to', 'to no revenue in') : statsCards.periodTooltip"
                            placement="right" data-toggle="tooltip">
                  <i class="fas fa-info-circle text-xs text-muted"></i>
                </el-tooltip>
              </span>
            </template>
          </stats-card>
        </div>
        <div class="col-12 col-sm-6 col-lg-3">
          <stats-card title="Total orders"
                      type="gradient-green"
                      :sub-title="statsCards.totalOrders"
                      :loading="statsCards.totalOrders == null"
                      icon="ni ni-cart">
            <template slot="footer">
              <span v-if="statsCards.totalOrdersDiff > 0" class="text-success mr-2" v-html="'<i class=\'fa fa-arrow-up\'></i> +' + statsCards.totalOrdersDiff + '%'"></span>
              <span v-if="statsCards.totalOrdersDiff === 0" class="mr-2" v-html="'0%'"></span>
              <span v-if="statsCards.totalOrdersDiff === 'inf'" class="mr-2" v-html="'+ <i class=\'fas fa-infinity text-muted\' style=\'margin-right: 2px;\'></i>%'"></span>
              <span v-if="statsCards.totalOrdersDiff < 0" class="text-danger mr-2" v-html="'<i class=\'fa fa-arrow-down\'></i> ' + statsCards.totalOrdersDiff + '%'"></span>
              <span class="text-nowrap">
                <el-tooltip :content="statsCards.totalOrdersDiff === 'inf' ? statsCards.periodTooltip.replace('to', 'to no orders in') : statsCards.periodTooltip"
                            placement="right" data-toggle="tooltip">
                  <i class="fas fa-info-circle text-xs text-muted"></i>
                </el-tooltip>
              </span>
            </template>
          </stats-card>
        </div>
        <div class="col-12 col-sm-6 col-lg-3">
          <stats-card title="Average order value"
                      type="gradient-info"
                      :sub-title="statsCards.averageOrderValue | restaurantCurrency"
                      :loading="statsCards.averageOrderValue == null"
                      icon="ni ni-tag">
            <template slot="footer">
              <span v-if="statsCards.averageOrderValueDiff > 0" class="text-success mr-2" v-html="'<i class=\'fa fa-arrow-up\'></i> +' + statsCards.averageOrderValueDiff + '%'"></span>
              <span v-if="statsCards.averageOrderValueDiff === 0" class="mr-2" v-html="'0%'"></span>
              <span v-if="statsCards.averageOrderValueDiff === 'inf'" class="mr-2" v-html="'+ <i class=\'fas fa-infinity text-muted\' style=\'margin-right: 2px;\'></i>%'"></span>
              <span v-if="statsCards.averageOrderValueDiff < 0" class="text-danger mr-2" v-html="'<i class=\'fa fa-arrow-down\'></i> ' + statsCards.averageOrderValueDiff + '%'"></span>
              <span class="text-nowrap">
                <el-tooltip :content="statsCards.averageOrderValueDiff === 'inf' ? statsCards.periodTooltip.replace('to', 'to no order value in') : statsCards.periodTooltip"
                            placement="right" data-toggle="tooltip">
                  <i class="fas fa-info-circle text-xs text-muted"></i>
                </el-tooltip>
              </span>
            </template>
          </stats-card>
        </div>
        <div class="col-12 col-sm-6 col-lg-3">
          <stats-card title="Total tips"
                      type="gradient-green"
                      :sub-title="statsCards.totalTips | restaurantCurrency"
                      :loading="statsCards.totalTips == null"
                      :tooltip="statsCards.totalTips ? (statsCards.averageTip + ' on average (over ' + statsCards.tipsCount + ' tips)') : ''"
                      icon="ni ni-satisfied">
            <template slot="footer">
              <span v-if="statsCards.totalTipsDiff > 0" class="text-success mr-2" v-html="'<i class=\'fa fa-arrow-up\'></i> +' + statsCards.totalTipsDiff + '%'"></span>
              <span v-if="statsCards.totalTipsDiff === 0" class="mr-2" v-html="'0%'"></span>
              <span v-if="statsCards.totalTipsDiff === 'inf'" class="mr-2" v-html="'+ <i class=\'fas fa-infinity text-muted\' style=\'margin-right: 2px;\'></i>%'"></span>
              <span v-if="statsCards.totalTipsDiff < 0" class="text-danger mr-2" v-html="'<i class=\'fa fa-arrow-down\'></i> ' + statsCards.totalTipsDiff + '%'"></span>
              <span class="text-nowrap">
                <el-tooltip :content="statsCards.totalTipsDiff === 'inf' ? statsCards.periodTooltip.replace('to', 'to no tips in') : statsCards.periodTooltip"
                            placement="right" data-toggle="tooltip">
                  <i class="fas fa-info-circle text-xs text-muted"></i>
                </el-tooltip>
              </span>
            </template>
          </stats-card>
        </div>
      </div>
    </div>

    <!--Charts-->
    <div class="container-fluid">
      <div class="row">
        <div class="col-xl-8">
          <card header-classes="bg-transparent">
            <div slot="header" class="row align-items-center">
              <div class="col-auto">
                <el-select v-model="bigLineChart.view" v-on:input="showRevenueChart(timeframe)"
                           filterable placeholder="View" class="float-left mr-2">
                  <el-option
                    v-for="view in Object.values(revenueChartViews)"
                    :key="view.value"
                    :label="view.label"
                    :value="view.value">
                    <span>{{ view.text }}</span>
                  </el-option>
                </el-select>

                <el-select v-model="bigLineChart.vat" v-on:input="showRevenueChart(timeframe)"
                           filterable placeholder="VAT" class="float-right">
                  <el-option
                    v-for="vat in [
                      {value: null, label: 'All orders', text: 'All orders'},
                      {value: 0.09, label: '9%', text: '9% VAT Products'},
                      {value: 0.21, label: '21%', text: '21% VAT Products'}
                    ]"
                    :key="vat.value"
                    :label="vat.label"
                    :value="vat.value">
                    <span>{{ vat.text }}</span>
                  </el-option>
                </el-select>
              </div>
            </div>
            <line-chart
              :key="bigLineChartKey"
              :height="350"
              ref="bigChartLine"
              :chart-data="bigLineChart.chartData"
              :extra-options="bigLineChartExtraOptions"
              v-if="bigLineChart.type === 'line'"
            >
            </line-chart>
            <bar-chart
              :key="bigLineChartKey"
              :height="350"
              ref="bigChartBar"
              :chart-data="bigLineChart.chartData"
              :extra-options="bigLineChartExtraOptions"
              v-if="bigLineChart.type === 'bar'"
            >
            </bar-chart>

          </card>
        </div>

        <div class="col-xl-4">
          <card header-classes="bg-transparent">
            <div slot="header" class="row align-items-center">
              <div class="col-auto">
                <h6 class="text-uppercase text-muted ls-1 mb-1">Performance</h6>
                <h5 class="h3 mb-0">Total orders</h5>
              </div>
              <div class="col">
              </div>
            </div>

            <bar-chart
              :height="350"
              ref="barChart"
              :chart-data="redBarChart.chartData"
              :extra-options="singleBarChart"
            >
            </bar-chart>
          </card>
        </div>
      </div>

    </div>

    <div class="container-fluid">
      <div class="row">

        <div class="col-xl-4">
          <card class="card-list-view" header-classes="bg-transparent">
            <div slot="header" class="row align-items-center">
              <div class="col-auto">
                <h6 class="text-uppercase text-muted ls-1 mb-1">Performance</h6>
                <h5 class="h3 mb-0">Meals</h5>
              </div>

              <div class="col-auto">
                <el-select v-model="tableFilter" v-on:input="showList('per_dish')"
                           clearable filterable placeholder="Table filter">
                  <el-option
                    v-for="table in allTables"
                    :key="table.value"
                    :label="table.value === null ? 'All tables' : table.label"
                    :value="table.value">
                    <span>{{ table.value === null ? 'All tables' : table.label }}</span>
                  </el-option>
                </el-select>
              </div>
            </div>

            <div v-if="revolists.per_dish == null" class="d-flex justify-content-center py-5">
              <div class="spinner-border text-primary" role="status">
                <span class="sr-only">Loading...</span>
              </div>
            </div>
            <div v-else-if="revolists.per_dish.length === 0" class="d-flex justify-content-center py-5">
              No data
            </div>
            <v-grid
              v-else
              id="revolist_per_dish"
              theme="compact"
              style="fontFamily: Open Sans, sans-serif; fontSize: .8125rem; height: 400px"
              :source="revolists.per_dish"
              :columns="columns('per_dish')"
              :filter="true"
              :resize="true"
              :readonly="true"
              :range="true">
            </v-grid>

          </card>
        </div>

        <div class="col-xl-4">
          <card class="card-list-view" header-classes="bg-transparent">
            <div slot="header" class="row align-items-center">
              <div class="col-auto">
                <h6 class="text-uppercase text-muted ls-1 mb-1">Performance</h6>
                <h5 class="h3 mb-0">Tables</h5>
              </div>
            </div>

            <div v-if="revolists.per_table == null" class="d-flex justify-content-center py-5">
              <div class="spinner-border text-primary" role="status">
                <span class="sr-only">Loading...</span>
              </div>
            </div>
            <div v-else-if="revolists.per_table.length === 0" class="d-flex justify-content-center py-5">
              No data
            </div>
            <v-grid
              v-else
              theme="compact"
              style="fontFamily: Open Sans, sans-serif; fontSize: .8125rem; height: 400px"
              :source="revolists.per_table"
              :columns="columns('per_table')"
              :filter="true"
              :resize="true"
              :readonly="true"
              :range="true">
            </v-grid>
          </card>
        </div>

        <div class="col-xl-4">
          <card class="card-list-view" header-classes="bg-transparent">
            <div slot="header" class="row align-items-center">
              <div class="col-auto">
                <h6 class="text-uppercase text-muted ls-1 mb-1">Performance</h6>
                <h5 class="h3 mb-0">Areas</h5>
              </div>
            </div>

            <div v-if="revolists.per_area == null" class="d-flex justify-content-center py-5">
              <div class="spinner-border text-primary" role="status">
                <span class="sr-only">Loading...</span>
              </div>
            </div>
            <div v-else-if="revolists.per_area.length === 0" class="d-flex justify-content-center py-5">
              No data
            </div>
            <v-grid
              v-else
              theme="compact"
              style="fontFamily: Open Sans, sans-serif; fontSize: .8125rem; height: 400px"
              :source="revolists.per_area"
              :columns="columns('per_area')"
              :filter="true"
              :resize="true"
              :readonly="true"
              :range="true">
            </v-grid>
          </card>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import DateRangePicker from "@/components/Vue2-daterange-picker/DateRangePicker.vue";
import VGrid, { VGridVueTemplate } from "@revolist/vue-datagrid";
import {vGridCurrency} from "@/views/Admin/vGridCurrency";
import {vGridPercentageChange} from "@/views/Admin/vGridPercentageChange";
import VGridDishName from "@/views/Admin/VGridDishName";
import StatsCard from "@/components/Cards/StatsCard";
import {addDays} from "@/filters/dateTime";
import {restaurantCurrency} from "@/filters/restaurantCurrency";

// Charts
import * as chartConfigs from '@/components/Charts/config';
import LineChart from '@/components/Charts/LineChart';
import BarChart from '@/components/Charts/BarChart';

export default {
  name: "Analytics",
  components: {
    LineChart,
    BarChart,
    VGrid,
    DateRangePicker,
    StatsCard,
  },
  data() {
    return {
      dateRange: {
        startDate: null,
        endDate: null,
      },
      bigLineChartKey: 0,
      singleBarChart: chartConfigs.singleBarChartOptions,
      bigLineChartExtraOptions: chartConfigs.lineChartOptionsCurrency,
      timeframe: 'D',
      statsCards: {
        totalRevenue: null,
        periodTooltip: null,
        totalOrders: null,
        averageOrderValue: null,
        totalTips: null,
      },
      revolists: {
        per_dish: null,
        per_table: null,
        per_area: null
      },
      tableFilter: null,
      allTables: [],
      bigLineChart: {
        type: 'line',
        timeframe: 'D',
        view: null,
        activeView: null,
        vat: null,
        chartData: {
          datasets: [
            {
              label: '',
              data: [],
            }
          ],
          labels: [],
        },
      },
      redBarChart: {
        timeframe: 'D',
        chartData: {
          datasets: [{
            label: '',
            data: [],
          }],
          labels: [],
        },
        extraOptions: chartConfigs.blueChartOptions
      },
      revenueChartViews: {
        null: {value: null, label: 'Total revenue', text: 'Total revenue', type: 'line'},
        per_area: {value: 'per_area', label: 'Revenue per area', text: 'Revenue per area', type: 'bar', legend_prefix: 'Area'},
        per_area_percentage: {value: 'per_area_percentage', label: 'Revenue % per area', text: 'Revenue % per area', type: 'bar', legend_prefix: 'Area'},
        per_table: {value: 'per_table', label: 'Revenue per table', text: 'Revenue per table', type: 'bar', legend_prefix: 'Table'},
        per_table_percentage: {value: 'per_table_percentage', label: 'Revenue % per table', text: 'Revenue % per table', type: 'bar', legend_prefix: 'Table'},
        per_item_type: {value: 'per_item_type', label: 'Revenue per type of item', text: 'Revenue per type of item', type: 'bar', legend_prefix: 'Item Type'},
        tips: {value: 'tips', label: 'Tips received', text: 'Tips received', type: 'line'},
        payments: {value: 'payments', label: 'Payments revenue', text: 'Payments received', type: 'line'}
      },
      allTimeframes: [
        {value: 'Y', label: 'Year'},
        {value: 'Q', label: 'Quarter'},
        {value: 'M', label: 'Month'},
        {value: 'W', label: 'Week'},
        {value: 'D', label: 'Day'},
        {value: 'H', label: 'Hour'}
      ],
      valuesPerTimeframe: {
        'H': {
          default: 12,
          chosen: null,
        },
        'D': {
          default: 14,
          chosen: null,
        },
        'W': {
          default: 20,
          chosen: null,
        },
        'M': {
          default: 12,
          chosen: null,
        },
        'Q': {
          default: 8,
          chosen: null,
        },
        'Y': {
          default: 5,
          chosen: null,
        }
      },

      activeColumns: {
        per_dish: ['dish_name', 'total', 'amount'],
        per_table: ['name', 'total', 'amount', 'average'],
        per_area: ['name', 'total', 'amount', 'average'],
      },
      columnsData: [{
          name: '',
          prop: 'name',
          pin: 'colPinStart',
          size: 120,
          sortable: true,
          cellProperties: ({prop, model}) => {
            return {
              title: model[prop],
            }
          }
        },
        {
          name: '',
          prop: 'dish_name',
          pin: 'colPinStart',
          size: 200,
          sortable: true,
          cellTemplate: this.isAdmin ? VGridVueTemplate(VGridDishName) : '',
          cellProperties: ({prop, model}) => {
            return {
              title: model[prop],
            }
          }
        },
        {
          name: 'Revenue',
          prop: 'total',
          size: 155,
          sortable: true,
          cellCompare: (prop, a, b) => {
            return (parseFloat(a[prop]) < parseFloat(b[prop])) ? -1 : 1;
          },
          cellTemplate: vGridPercentageChange,
          cellProperties: ({prop, model}) => {
            return {
              title: model['total_title'],
            }
          }
        },
        {
          name: 'Orders',
          prop: 'amount',
          size: 155,
          sortable: true,
          cellCompare: (prop, a, b) => {
            return (parseInt(a[prop]) < parseInt(b[prop])) ? -1 : 1;
          },
          cellTemplate: vGridPercentageChange,
          cellProperties: ({prop, model}) => {
            return {
              title: model['amount_title'],
            }
          }
        },
        {
          name: 'Average order',
          prop: 'average',
          size: 150,
          sortable: true,
          cellCompare: (prop, a, b) => {
            return (parseFloat(a[prop]) < parseFloat(b[prop])) ? -1 : 1;
          },
          cellTemplate: vGridCurrency,
          cellProperties: ({prop, model}) => {
            return {
              title: window.Vue.filter('restaurantCurrency')(model[prop]) + ' average order value',
            }
          }
        }
      ]
    };
  },
  computed: {
    ranges() {
      // Today
      let today = new Date()

      today.setHours(0, 0, 0, 0)
      let dayOfWeek = today.getDay();

      // Yesterday
      let yesterday = new Date(today)
      yesterday.setDate(today.getDate() - 1)

      // This week
      let thisMonday = new Date(today.getFullYear(), today.getMonth(), today.getDate() - (dayOfWeek + 6) % 7);

      // Last week
      let lastWeekStart = new Date(today.getFullYear(), today.getMonth(), today.getDate()   -(dayOfWeek % 7 || 7) - 6); // Get the start date of last week
      let lastWeekEnd = new Date(today.getFullYear(), today.getMonth(), today.getDate() - (dayOfWeek % 7 || 7)); // Get the end date of last week

      // This month (x)
      let beginThisMonth = new Date(today.getFullYear(), today.getMonth(), 1)
      let thisMonth = beginThisMonth.toLocaleString('default', { month: 'short' });

      // Last month (x)
      let beginLastMonth = new Date(today.getFullYear(), today.getMonth() - 1, 1)
      let endLastMonth = new Date(today.getFullYear(), today.getMonth(), 0)
      let lastMonth = beginLastMonth.toLocaleString('default', { month: 'short' });

      // This year (x)
      let beginThisYear = new Date(today.getFullYear(), 0, 1)
      let thisYear = beginThisYear.getFullYear();

      // Last year (x)
      let beginLastYear = new Date(today.getFullYear() -1, 0, 1)
      let endLastYear = new Date(today.getFullYear() -1, 11, 31)
      let lastYear = beginLastYear.getFullYear();

      return {
        'Today': [today, today],
        'Yesterday': [yesterday, yesterday],
        'This week': [thisMonday, today],
        'Last week': [lastWeekStart, lastWeekEnd],
        [`This month (${thisMonth})`]: [beginThisMonth, today],
        [`Last month (${lastMonth})`]: [beginLastMonth, endLastMonth],
        [`This year (${thisYear})`]: [beginThisYear, today],
        [`Last year (${lastYear})`]: [beginLastYear, endLastYear],
      }
    }
  },
  watch: {
    '$route.query.startDate'(newValue, oldValue) {
      if (newValue && this.dateString(this.dateRange.startDate) !== newValue) {
        this.dateRange.startDate = new Date(this.$route.query.startDate);
        this.dateRange.endDate = new Date(this.$route.query.endDate);
        this.timeframe = this.$route.query.groupBy;
        this.reloadAnalytics();
      }
    },
    '$route.query.endDate'(newValue, oldValue) {
      if (newValue && this.dateString(this.dateRange.endDate) !== newValue) {
        this.dateRange.startDate = new Date(this.$route.query.startDate);
        this.dateRange.endDate = new Date(this.$route.query.endDate);
        this.timeframe = this.$route.query.groupBy;
        this.reloadAnalytics();
      }
    },
    '$route.query.groupBy'(newValue, oldValue) {
      if (newValue && this.timeframe !== newValue) {
        this.dateRange.startDate = new Date(this.$route.query.startDate);
        this.dateRange.endDate = new Date(this.$route.query.endDate);
        this.timeframe = this.$route.query.groupBy;
        this.reloadAnalytics(false);
      }
    }
  },
  methods: {
    reloadAnalytics(fullReload = true) {
      // remove time from rangepicker enddate (else current year/month range will end at current time)
      this.dateRange.endDate = new Date(Date.UTC(
        this.dateRange.endDate.getFullYear(),
        this.dateRange.endDate.getMonth(),
        this.dateRange.endDate.getDate())
      );
      this.showRevenueChart(this.timeframe);
      this.showOrdersChart(this.timeframe);
      if (fullReload) {
        this.showStatsCards();
        this.showList("per_dish");
        this.showList("per_table");
        this.showList("per_area");
      }
      // update URL parameters if needed
      if (this.$route.query.startDate !== this.dateString(this.dateRange.startDate) ||
        this.$route.query.endDate !== this.dateString(this.dateRange.endDate) ||
        this.$route.query.groupBy !== this.timeframe) {
        this.$router.push({path: "/analytics", query: {
          startDate: this.dateString(this.dateRange.startDate),
          endDate: this.dateString(this.dateRange.endDate),
          groupBy: this.timeframe
        }})
      }
    },
    showStatsCards() {
      this.statsCards = {
        totalRevenue: null,
        totalOrders: null,
        averageOrderValue: null,
        totalTips: null,
      }
      let self = this;
      let payload = {
        startDate: this.dateString(this.dateRange.startDate),
        endDate: this.dateString(this.dateRange.endDate),
      };
      this.$store.dispatch('getStatsOfRestaurant', payload)
        .then(function (res) {
          let statistics = res.data.results;
          self.statsCards.periodTooltip = 'Compared to ' + statistics.previous.start_date + " till " + statistics.previous.end_date;

          self.statsCards.totalRevenue = statistics.total_revenue;
          if (statistics.previous.total_revenue === 0) {
            self.statsCards.totalRevenueDiff = statistics.total_revenue > 0 ? 'inf' : 0;
          } else {
            self.statsCards.totalRevenueDiff = (statistics.total_revenue / statistics.previous.total_revenue * 100) - 100;
            self.statsCards.totalRevenueDiff = Math.round(self.statsCards.totalRevenueDiff * 100) / 100;
          }

          self.statsCards.totalOrders = statistics.total_orders.toLocaleString();
          if (statistics.previous.total_orders === 0) {
            self.statsCards.totalOrdersDiff = statistics.total_orders > 0 ? 'inf' : 0;
          } else {
            self.statsCards.totalOrdersDiff = (statistics.total_orders / statistics.previous.total_orders * 100) - 100;
            self.statsCards.totalOrdersDiff = Math.round(self.statsCards.totalOrdersDiff * 100) / 100;
          }

          self.statsCards.averageOrderValue = statistics.average_order_value;
          if (statistics.previous.average_order_value === 0) {
            self.statsCards.averageOrderValueDiff = statistics.average_order_value > 0 ? 'inf' : 0;
          } else {
            self.statsCards.averageOrderValueDiff = (statistics.average_order_value / statistics.previous.average_order_value * 100) - 100;
            self.statsCards.averageOrderValueDiff = Math.round(self.statsCards.averageOrderValueDiff * 100) / 100;
          }

          self.statsCards.totalTips = statistics.total_tips;
          self.statsCards.tipsCount = statistics.tips_count;
          self.statsCards.averageTip = restaurantCurrency(0);
          if (statistics.tips_count > 0) {
            self.statsCards.averageTip = restaurantCurrency(statistics.total_tips / statistics.tips_count);
          }
          if (statistics.previous.total_tips === 0) {
            self.statsCards.totalTipsDiff = statistics.tips_count > 0 ? 'inf' : 0;
          } else {
            self.statsCards.totalTipsDiff = (statistics.total_tips / statistics.previous.total_tips * 100) - 100;
            self.statsCards.totalTipsDiff = Math.round(self.statsCards.totalTipsDiff * 100) / 100;
          }
        })
        .catch(function (err) {
          alert(err)
        })
    },
    showList(view) {
      let self = this;
      this.revolists[view] = null;
      let startDateString = this.dateString(this.dateRange.startDate);
      let endDateString = this.dateString(this.dateRange.endDate);
      let payload = {
        view: view,
        tableFilter: this.tableFilter,
        startDate: startDateString,
        endDate: endDateString,
      };
      this.$store.dispatch('getRevenueListOfRestaurant', payload)
        .then(function (res) {
          if (res.data.results.length === 0) {
            self.revolists[view] = [];
            return;
          }

          if (view === 'per_table') {
            self.allTables = [];
          }
          let namePrefix = '';
          if (view in self.revenueChartViews) {
            namePrefix = self.revenueChartViews[view].legend_prefix + ' ';
          }
          let entries = []
          res.data.results.forEach(x => {
            if (view === 'per_table') {
              self.allTables.push({value: x[0], label: namePrefix + x[1]});
            }
            let entry = {
              id: x[0],
              name: namePrefix + x[1],
              dish_name: namePrefix + x[1],
              start_date: startDateString,
              end_date: endDateString,
              total: x[2],
              total_title: window.Vue.filter('restaurantCurrency')(x[2]),
              amount: x[3],
              amount_title: x[3] + ' orders',
              average: Math.round(x[2] / x[3] * 100) / 100,
            }
            entries.push(entry)
          });
          self.allTables.sort((a, b) => a.value - b.value);

          // load differences with previous period
          let duration = self.daysBetween(self.dateRange.startDate, self.dateRange.endDate);
          let payload = {
            view: view,
            tableFilter: self.tableFilter,
            startDate: self.dateString(self.addDays(startDateString, -duration)),
            endDate: startDateString,
          };
          self.$store.dispatch('getRevenueListOfRestaurant', payload)
            .then(function (res) {
              let previousTotals = {};
              let previousAmounts = {};
              res.data.results.forEach(x => {
                previousTotals[x[0]] = x[2];
                previousAmounts[x[0]] = x[3];
              });

              let hue;
              let hueMid = 60;
              let hueMax = 125
              let newEntries = [];
              self.revolists[view].forEach(entry => {
                entry.total_percentage = null;
                entry.amount_percentage = null;
                if (entry.id in previousTotals) {
                  let previousTotal = previousTotals[entry.id];
                  let totalPercentage = (entry.total / previousTotal * 100) - 100;
                  if (! isNaN(totalPercentage)) {
                    entry.total_percentage = totalPercentage;
                    entry.total_title += ' (compared to ' + restaurantCurrency(previousTotal) + ' from ' + payload.startDate + ' till ' + payload.endDate + ')';
                    if (totalPercentage >= 0) {
                      hue = Math.min(hueMid + (totalPercentage * (hueMax - hueMid) / 100), hueMax);
                    } else {
                      hue = Math.max(((100 + totalPercentage) / 100) * hueMid, 0);
                    }
                    entry.total_color = 'hsl(' + hue + ', 68.5%, 49.8%)';
                  }

                  let previousAmount = previousAmounts[entry.id];
                  let amountPercentage = (entry.amount / previousAmount * 100) - 100;
                  if (! isNaN(amountPercentage)) {
                    entry.amount_percentage = amountPercentage;
                    entry.amount_title = 'Compared to ' + previousAmount + ' orders from ' + payload.startDate + ' till ' + payload.endDate + '';
                    if (amountPercentage >= 0) {
                      hue = Math.min(hueMid + (amountPercentage * (hueMax - hueMid) / 100), hueMax);
                    } else {
                      hue = Math.max(((100 + amountPercentage) / 100) * hueMid, 0);
                    }
                    entry.amount_color = 'hsl(' + hue + ', 68.5%, 49.8%)';
                  }
                } else {
                  if (entry.total > 0) {
                    entry.total_percentage = 'inf';
                    entry.total_color = 'hsl(' + hueMax + ', 68.5%, 49.8%)';
                    entry.total_title += ' (compared to no revenue from ' + payload.startDate + ' till ' + payload.endDate + ')';
                  }
                  if (entry.amount > 0) {
                    entry.amount_percentage = 'inf';
                    entry.amount_color = 'hsl(' + hueMax + ', 68.5%, 49.8%)';
                    entry.amount_title += ' (compared to no orders from ' + payload.startDate + ' till ' + payload.endDate + ')';
                  }
                }
                newEntries.push(entry);
              });
              self.revolists[view] = newEntries;
            });
          self.revolists[view] = entries;
        })
        .catch(function (err) {
          alert(err)
        })
    },
    showRevenueChart(timeframe) {
      let self = this;
      let payload = {
        timeframeKey: timeframe,
        vat: this.bigLineChart.vat,
        view: this.bigLineChart.view ? this.bigLineChart.view.replace('_percentage', '') : null,
        startDate: this.dateString(this.dateRange.startDate),
        endDate: this.dateString(this.dateRange.endDate),
      };
      let hasPercentageView = self.bigLineChart.view && self.bigLineChart.view.includes('_percentage');
      let revenueChartView = this.revenueChartViews[this.bigLineChart.view];

      if (this.bigLineChart.view === 'payments') {
        this.$store.dispatch('getPaymentsRevenue', payload)
          .then(function (res) {
            let amountOfResultsToShow = self.valuesPerTimeframe[timeframe];
            let results = self.populateEmptyValues(timeframe, amountOfResultsToShow.chosen ? amountOfResultsToShow.chosen : amountOfResultsToShow.default, res.data.results)

            let datasets = []
            let labels = [];

            self.bigLineChart.type = revenueChartView.type;
            self.bigLineChart.activeView = self.bigLineChart.view;

            if (Array.isArray(results)) {
              // chart with 1 dataset
              let data = [];
              results.forEach(x => {
                data.push(Object.values(x)[0]);
                labels.push(Object.keys(x)[0])
              });
              labels = labels.map(date => new Date(date.replace(/ /g, "T") + "Z"));
              labels = self.labelsBy(timeframe, labels);
              datasets.push({
                label: 'Revenue',
                data: data
              });
            }

            self.bigLineChart.chartData = {
              datasets: datasets,
              labels: labels,
            };
            self.bigLineChart.timeframe = timeframe;
            if (!self.bigLineChartKey) self.bigLineChartKey++; //Bugfix for chart not showing currency on y-axis because API data doens't exist on load. Component key is initialized at 0, and only changed after the first server call which reloads the component
          })
          .catch(function (err) {

          })
      } else {
        this.$store.dispatch('getRevenueOfRestaurant', payload)
          .then(function (res) {
            let amountOfResultsToShow = self.valuesPerTimeframe[timeframe];
            let results = self.populateEmptyValues(timeframe, amountOfResultsToShow.chosen ? amountOfResultsToShow.chosen : amountOfResultsToShow.default, res.data.results)

            let datasets = []
            let labels = [];

            self.bigLineChart.type = revenueChartView.type;
            self.bigLineChart.activeView = self.bigLineChart.view;

            if (Array.isArray(results)) {
              // chart with 1 dataset
              let data = [];
              results.forEach(x => {
                data.push(Object.values(x)[0]);
                labels.push(Object.keys(x)[0])
              });
              labels = labels.map(date => new Date(date.replace(/ /g, "T") + "Z"));
              labels = self.labelsBy(timeframe, labels);
              datasets.push({
                label: 'Revenue',
                data: data
              });

            } else {
              // chart with multiple datasets
              let totalPerDay = []
              self.bigLineChartExtraOptions = chartConfigs.lineChartOptionsCurrency;
              if (hasPercentageView) {
                self.bigLineChartExtraOptions = chartConfigs.chartOptionsPercentage;
                for (const [subChartKey, subChartData] of Object.entries(results)) {
                  let i = 0;
                  subChartData.forEach(x => {
                    totalPerDay[i] = (totalPerDay[i] ? totalPerDay[i] : 0) + Object.values(x)[0];
                    i++;
                  });
                }
              }

              let datasetsMaximum = 0;
              for (const [subChartKey, subChartData] of Object.entries(results)) {
                let data = [];
                subChartData.forEach(x => {
                  data.push(Object.values(x)[0]);
                  if (datasets.length === 0) {
                    labels.push(Object.keys(x)[0])
                  }
                });
                if (datasets.length === 0) {
                  labels = labels.map(date => new Date(date.replace(/ /g, "T") + "Z"));
                  labels = self.labelsBy(timeframe, labels);
                }
                if (data.length > 0) {
                  let label = revenueChartView.legend_prefix + ' ' + subChartKey
                  if (subChartKey === 'null') {
                    label = 'Unknown ' + revenueChartView.legend_prefix.toLowerCase();
                  }
                  let datasetSum = data.reduce((a, b) => (a + b));
                  datasetsMaximum = Math.max(datasetsMaximum, datasetSum)
                  if (hasPercentageView) {
                    data = data.map(function(n, i) { return Math.round(n / totalPerDay[i] * 1000) / 10; });
                  }
                  datasets.push({
                    label: label,
                    data: data,
                    sum: datasetSum
                  });
                }
              }
              datasets.forEach(dataset => {
                let hueMax = 125
                let hue = dataset.sum * hueMax / datasetsMaximum;
                dataset.backgroundColor = 'hsl(' + hue + ', 68.5%, 49.8%)'
              })
            }

            self.bigLineChart.chartData = {
              datasets: datasets,
              labels: labels,
            };
            self.bigLineChart.timeframe = timeframe;
            if (!self.bigLineChartKey) self.bigLineChartKey++; //Bugfix for chart not showing currency on y-axis because API data doens't exist on load. Component key is initialized at 0, and only changed after the first server call which reloads the component
          })
          .catch(function (err) {
            alert(err)
          })
      }
    },
    populateEmptyValues(timeframe, amount, apiData) {
      //Function take a timeframe (e.g. H hour, D day) and a number of datetimes it should return, then returns an object of all possible times with default value 0 and api results overwritten

      let dataWithZero = [];
      let newestTimeValue = new Date();
      let hoursPerInterval = 0;
      amount = this.daysBetween(this.dateRange.startDate, this.dateRange.endDate) + 1;

      function previousTimeFn() {
      }

      switch (timeframe) {
        case "Y":
          //Chart is by year
          newestTimeValue = new Date(Date.UTC(newestTimeValue.getFullYear(), 0));
          previousTimeFn = function () {
            newestTimeValue.setFullYear(newestTimeValue.getFullYear() - 1)
          }
          break;

        case "Q":
          //Chart is by quarter, round the month down to a multiple of 3 (month 0 is january, so start start of Q1 || month 3 is april, so start of Q2 || etc.)
          newestTimeValue = new Date(Date.UTC(newestTimeValue.getFullYear(), Math.floor(newestTimeValue.getMonth() / 3) * 3, 1));
          previousTimeFn = function () {
            newestTimeValue = new Date(Date.UTC(newestTimeValue.getFullYear(), newestTimeValue.getMonth() - 3, 1))
          }
          break;

        case "M":
          //Chart is by month
          newestTimeValue = new Date(Date.UTC(newestTimeValue.getFullYear(), newestTimeValue.getMonth(), 1)); // newestTimeValue.setDate(1) zou ook moeten werken
          previousTimeFn = function () {
            newestTimeValue = new Date(Date.UTC(newestTimeValue.getFullYear(), newestTimeValue.getMonth() - 1, 1))
          }
          break;

        case "W":
          //Chart is by week, round current time down to a week
          newestTimeValue = new Date(this.dateRange.endDate);
          amount /= 7;
          let newestDateWeekday = newestTimeValue.getDay() || 7; // Get current day number, converting Sun. to 7
          if (newestDateWeekday !== 1) {               // Only manipulate the date if it isn't Mon.
            newestTimeValue.setDate(newestTimeValue.getDate() - (newestDateWeekday - 1));   // Remove the number of days untill it's a monday
          }
          newestTimeValue = new Date(Date.UTC(newestTimeValue.getFullYear(), newestTimeValue.getMonth(), newestTimeValue.getDate())); //Lastly set hours,min,sec,ms to 0 in UTC time. This is done last as the day might be different in UTC than in the users timezone
          // previousTimeFn = function() {newestTimeValue.setDate(newestTimeValue.getDate() - 7)}
          previousTimeFn = function () {
            newestTimeValue = new Date(Date.UTC(newestTimeValue.getFullYear(), newestTimeValue.getMonth(), newestTimeValue.getDate() - 7))
          }
          break;

        case "D":
          //Chart is by day, round current time down to a day
          newestTimeValue = new Date(this.dateRange.endDate);
          previousTimeFn = function () {
            newestTimeValue = new Date(Date.UTC(newestTimeValue.getFullYear(), newestTimeValue.getMonth(), newestTimeValue.getDate() - 1))
          }
          break;

        case "H":
          //Chart is by hour, round current time down to an hour
          newestTimeValue = new Date(this.dateRange.endDate);
          newestTimeValue.setMinutes(0, 0, 0);
          previousTimeFn = function () {
            newestTimeValue.setHours(newestTimeValue.getHours() - 1)
          }
          break;

        default:
          console.log('switch was default');
      }

      for (let i = 0; i < amount; i++) {
        let timeString = newestTimeValue.toISOString().split('T')[0] + ' ' + newestTimeValue.toISOString().split('T')[1].slice(0, 8)
        dataWithZero.push({[timeString]: 0})
        previousTimeFn();
        // newestTimeValue.setHours(newestTimeValue.getHours() - hoursPerInterval)
      }

      if (Array.isArray(apiData)) {
        apiData.forEach(x => {
          let targetTime = dataWithZero.find(y => {
            return Object.keys(y)[0] === Object.keys(x)[0]
          })
          if (targetTime) Object.assign(targetTime, x)
        })
        dataWithZero.reverse();
        // console.log(apiData);
        // console.log(dataWithZero);
        return dataWithZero;
      } else {
        let multiChartData = {};
        let originalDataWithZero = JSON.parse(JSON.stringify(dataWithZero));
        for (const [subChartKey, subChartData] of Object.entries(apiData)) {
          dataWithZero = JSON.parse(JSON.stringify(originalDataWithZero));
          subChartData.forEach(x => {
            let targetTime = dataWithZero.find(y => {
              return Object.keys(y)[0] === Object.keys(x)[0]
            })
            if (targetTime) Object.assign(targetTime, x)
          })
          dataWithZero.reverse();
          multiChartData[subChartKey] = dataWithZero;
        }
        return multiChartData;
      }
    },
    showOrdersChart(timeframe) {
      let self = this;
      let payload = {
        timeframeKey: timeframe,
        startDate: this.dateString(this.dateRange.startDate),
        endDate: this.dateString(this.dateRange.endDate),
      };
      this.$store.dispatch('getOrdersOfRestaurant', payload)
        .then(function (res) {
          let amountOfResultsToShow = self.valuesPerTimeframe[timeframe];
          let results = self.populateEmptyValues(timeframe, amountOfResultsToShow.chosen ? amountOfResultsToShow.chosen : Math.round(amountOfResultsToShow.default / 2), res.data.results)

          let data = [];
          let labels = [];
          results.forEach(x => {
            data.push(Object.values(x)[0]);
            labels.push(Object.keys(x)[0])
          });
          labels = labels.map(date => new Date(date.replace(/ /g, "T") + "Z"));
          labels = self.labelsBy(timeframe, labels);
          self.redBarChart.timeframe = timeframe;
          self.redBarChart.chartData = {
            datasets: [
              {
                label: 'Orders',
                data: data
              }
            ],
            labels: labels,
          };
          // this.bigLineChartKey++;
        })
        .catch(function (err) {
          alert(err)
        })
    },
    labelsBy(timeframe, labels) {
      switch (timeframe) {
        case "Y":
          //Chart is by year
          return labels.map(dateObj => {
            return dateObj.toLocaleString([], {year: 'numeric'});
          });

        case "Q":
          //Chart is by quarter
          return labels.map(dateObj => {

            return 'Q' + Math.ceil((dateObj.getMonth() + 1) / 3) + ' ' + dateObj.toLocaleString([], {year: 'numeric'});
          });

        case "M":
          //Chart is by month
          return labels.map(dateObj => {
            return dateObj.toLocaleString([], {month: 'short', year: 'numeric'});
          });

        case "W":
          //Chart is by week
          return labels.map(dateObj => {
            var date = new Date(dateObj.getTime());
            date.setHours(0, 0, 0, 0);
            // Thursday in current week decides the year.
            date.setDate(date.getDate() + 3 - (date.getDay() + 6) % 7);
            // January 4 is always in week 1.
            var week1 = new Date(date.getFullYear(), 0, 4);
            // Adjust to Thursday in week 1 and count number of weeks from date to week1.
            return 1 + Math.round(((date.getTime() - week1.getTime()) / 86400000
              - 3 + (week1.getDay() + 6) % 7) / 7);
          });

        case "D":
          //Chart is by day
          return labels.map(dateObj => {
            return dateObj.toLocaleString([], {weekday: 'short', day: 'numeric', month: 'short'});
          });

        case "H":
          //Chart is by hour
          return labels.map(dateObj => {
            let start = dateObj.toLocaleTimeString([], {hour: '2-digit', minute: '2-digit'});
            dateObj.setHours(dateObj.getHours() + 1);
            let end = dateObj.toLocaleTimeString([], {hour: '2-digit', minute: '2-digit'});
            return start + " - " + end;
          });

        default:
          console.log('switch was default');
          return labels;
      }
    },
    datesSelected(selectedDates) {
      let self = this;
      // console.log(selectedDates[0]);
      // console.log(selectedDates[0].toISOString());
      // console.log(selectedDates[1]);
      // console.log(selectedDates[1].toISOString());
      // const startDate = selectedDates[0].toISOString().slice(0, 10);
      // const endDate = selectedDates[1].toISOString().slice(0, 10);
      const startDate = selectedDates[0].toLocaleString('sv', {timeZoneName: 'short'}).slice(0, 10) + " 00:00:00";
      let nextDay = new Date(selectedDates[1]);
      nextDay.setDate(selectedDates[1].getDate() + 1);
      const endDate = nextDay.toLocaleString('sv', {timeZoneName: 'short'}).slice(0, 10) + " 00:00:00";
      let dateRange = {start: startDate, end: endDate};

      // self.loading = true;
      // this.$store.dispatch('getAllRestaurantRevenue', dateRange)
      //   .then(function(res){
      //     self.loading = false;
      //   })
      //   .catch(function(err){
      //     alert(err);
      //   })
      //
      // const dateDiff = selectedDates[1] - selectedDates[0];
      // const prevStartDate = new Date(selectedDates[0] - dateDiff).toLocaleString( 'sv', { timeZoneName: 'short' } ).slice(0, 10);
      // const prevDateRange = {start: prevStartDate, end: startDate}
      // this.$store.dispatch('getPrevAllRestaurantRevenue', prevDateRange)
      //   .then(function(res){
      //     self.loading = false;
      //   })
      //   .catch(function(err){
      //     alert(err);
      //   })
    },
    columns(type) {
      return this.columnsData.filter(column => this.activeColumns[type].includes(column.prop));
    },
    isAdmin() {
      return this.$store.getters.isAdmin;
    }
  },
  mounted() {
    if (this.$route.query.groupBy) {
      this.timeframe = this.$route.query.groupBy;
    }

    this.dateRange.startDate = this.$route.query.startDate;
    this.dateRange.endDate = this.$route.query.endDate;
    if (! this.dateRange.startDate || ! this.dateRange.endDate) {
      let date = new Date();
      this.dateRange.endDate = new Date(Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()));
      this.dateRange.startDate = addDays(this.dateRange.endDate, -30);

      // for local development
      if (window.location.toString().includes("localhost")) {
        this.dateRange.startDate = new Date(Date.UTC(2021, 4, 1));
        this.dateRange.endDate = new Date(Date.UTC(2021, 5, 1));
      }
    }
    this.dateRange.startDate = new Date(this.dateRange.startDate);
    this.dateRange.endDate = new Date(this.dateRange.endDate);

    this.reloadAnalytics();
  }
}
</script>
