<template>
  <data-view
    :title="title"
    :title-id="titleId"
    :date="date"
    :head-title="title + infoTitles.join(',')"
  >
    <ul
      :class="$style.GraphLegend"
      :style="{ display: canvas ? 'block' : 'none' }"
    >
      <li
        v-for="(dataLabel, i) in dataLabels"
        :key="i"
        @click="onClickLegend(i)"
      >
        <button>
          <div
            v-if="i === 4"
            :style="{
              background: `repeating-linear-gradient(90deg, ${colors[i].fillColor}, ${colors[i].fillColor} 2px, #fff 2px, #fff 4px)`,
              border: 0,
              height: '3px',
            }"
          />
          <div
            v-else-if="i === 5"
            :style="{
              backgroundColor: colors[4].fillColor,
              border: 0,
              height: '3px',
            }"
          />
          <div
            v-else
            :style="{
              backgroundColor: colors[i].fillColor,
              borderColor: colors[i].strokeColor,
            }"
          />
          <span
            :style="{
              textDecoration: displayLegends[i] ? 'none' : 'line-through',
            }"
            >{{ dataLabel }}</span
          >
        </button>
      </li>
    </ul>
    <h4 :id="`${titleId}-graph`" class="visually-hidden">
      {{ $t(`{title}のグラフ`, { title }) }}
    </h4>
    <scrollable-chart v-show="canvas" :display-data="displayData">
      <template v-slot:chart="{ chartWidth }">
        <bar
          :ref="'barChart'"
          :chart-id="chartId"
          :chart-data="displayData"
          :options="displayOption"
          :display-legends="displayLegends"
          :height="240"
          :width="chartWidth"
        />
      </template>
      <template v-slot:sticky-chart>
        <bar
          class="sticky-legend"
          :chart-id="`${chartId}-header-right`"
          :chart-data="displayDataHeader"
          :options="displayOptionHeader"
          :plugins="yAxesBgRightPlugin"
          :display-legends="displayLegends"
          :height="240"
        />
      </template>
    </scrollable-chart>
    <template v-slot:additionalDescription>
      <slot name="additionalDescription" />
    </template>
    <template v-slot:dataTable>
      <client-only>
        <data-view-table :headers="tableHeaders" :items="tableData" />
      </client-only>
    </template>
    <template v-slot:dataSetPanel>
      <data-view-data-set-panel
        :title="infoTitles[0]"
        :l-text="displayInfo[0].lText"
        :s-text="displayInfo[0].sText"
        :s-text-under="displayInfo[0].sTextUnder"
        :unit="displayInfo[0].unit"
      />
      <data-view-data-set-panel
        :title="infoTitles[1]"
        :l-text="displayInfo[1].lText"
        :s-text="displayInfo[1].sText"
        :s-text-under="displayInfo[1].sTextUnder"
        :unit="displayInfo[1].unit"
      />
    </template>
  </data-view>
</template>

<script lang="ts">
import Vue from 'vue'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'
import { TranslateResult } from 'vue-i18n'
import { Chart } from 'chart.js'
import dayjs from 'dayjs'
import DataView from '@/components/DataView.vue'
import DataViewTable, {
  TableHeader,
  TableItem,
} from '@/components/DataViewTable.vue'
import DataViewDataSetPanel from '@/components/DataViewDataSetPanel.vue'
import ScrollableChart from '@/components/ScrollableChart.vue'
import {
  DisplayData,
  yAxesBgPlugin,
  yAxesBgRightPlugin,
} from '@/plugins/vue-chart'
import {
  getGraphSeriesStyle,
  getGraphSeriesColor,
  SurfaceStyle,
} from '@/utils/colors'
import { calcDayBeforeRatio } from '@/utils/formatDayBeforeRatio'
import { getNumberToFixedFunction } from '@/utils/monitoringStatusValueFormatters'

interface HTMLElementEvent<T extends HTMLElement> extends MouseEvent {
  currentTarget: T
}
type Data = {
  canvas: boolean
  displayLegends: boolean[]
  colors: SurfaceStyle[]
}
type Methods = {
  onClickLegend: (i: number) => void
}
type DisplayInfo = {
  lText: string
  sText: string
  sTextUnder: string
  unit: string
}
type Computed = {
  displayInfo: DisplayInfo[]
  displayData: DisplayData
  displayOption: Chart.ChartOptions
  displayDataHeader: DisplayData
  displayOptionHeader: Chart.ChartOptions
  scaledTicksYAxisMax: number
  scaledTicksYAxisMaxRight: number
  tableHeaders: TableHeader[]
  tableData: TableItem[]
}

type Props = {
  title: string
  titleId: string
  infoTitles: string[]
  chartId: string
  chartData: number[][]
  getFormatter: Function
  date: string
  labels: string[]
  dataLabels: string[] | TranslateResult[]
  tableLabels: string[] | TranslateResult[]
  unit: string
  optionUnit: string
  yAxesBgPlugin: Chart.PluginServiceRegistrationOptions[]
  yAxesBgRightPlugin: Chart.PluginServiceRegistrationOptions[]
}

const options: ThisTypedComponentOptionsWithRecordProps<
  Vue,
  Data,
  Methods,
  Computed,
  Props
> = {
  created() {
    this.canvas = process.browser
  },
  components: {
    DataView,
    DataViewTable,
    DataViewDataSetPanel,
    ScrollableChart,
  },
  props: {
    title: {
      type: String,
      default: '',
    },
    titleId: {
      type: String,
      required: false,
      default: '',
    },
    infoTitles: {
      type: Array,
      required: false,
      default: () => [],
    },
    chartId: {
      type: String,
      default: 'PositiveRateMixedChart',
    },
    chartData: {
      type: Array,
      required: false,
      default: () => [],
    },
    getFormatter: {
      type: Function,
      required: false,
      default: (_: number) => getNumberToFixedFunction(),
    },
    date: {
      type: String,
      required: true,
      default: '',
    },
    labels: {
      type: Array,
      default: () => [],
    },
    dataLabels: {
      type: Array,
      default: () => [],
    },
    tableLabels: {
      type: Array,
      default: () => [],
    },
    unit: {
      type: String,
      default: '',
    },
    optionUnit: {
      type: String,
      required: false,
      default: '',
    },
    yAxesBgPlugin: {
      type: Array,
      default: () => yAxesBgPlugin,
    },
    yAxesBgRightPlugin: {
      type: Array,
      default: () => yAxesBgRightPlugin,
    },
  },
  data: () => ({
    displayLegends: [true, true, true, true, true, true],
    colors: [...getGraphSeriesStyle(4), getGraphSeriesColor('E')],
    canvas: true,
  }),
  computed: {
    displayInfo() {
      const { lastDay, lastDayData, dayBeforeRatio } = calcDayBeforeRatio({
        displayData: this.displayData,
        dataIndex: 5,
        digit: 1,
      })
      const {
        lastDay: lastDay4,
        lastDayData: lastDayData4,
        dayBeforeRatio: dayBeforeRatio4,
      } = calcDayBeforeRatio({
        displayData: this.displayData,
        dataIndex: 4,
        digit: 1,
      })

      return [
        {
          lText: lastDayData,
          sText: `${this.$t('{date} の数値', {
            date: lastDay,
          })}（${this.$t('７日間移動平均値をもとに算出')}）`,
          sTextUnder: `（${this.$t('前日比')}: ${dayBeforeRatio} ${
            this.unit
          }）`,
          unit: this.unit,
        },
        {
          lText: lastDayData4,
          sText: `${this.$t('{date} の数値', {
            date: lastDay4,
          })}（${this.$t('７日間移動平均')}）`,
          sTextUnder: `（${this.$t('前日比')}: ${dayBeforeRatio4} ${
            this.optionUnit
          }）`,
          unit: this.optionUnit,
        },
      ]
    },
    displayData() {
      const graphSeries = [...getGraphSeriesStyle(4), getGraphSeriesColor('E')]
      return {
        labels: this.labels,
        datasets: [
          {
            type: 'bar',
            yAxisID: 'y-axis-1',
            label: this.dataLabels[0],
            data: this.chartData[0],
            backgroundColor: graphSeries[0].fillColor,
            borderColor: graphSeries[0].strokeColor,
            borderWidth: 1,
            order: 1,
          },
          {
            type: 'bar',
            yAxisID: 'y-axis-1',
            label: this.dataLabels[1],
            data: this.chartData[1],
            backgroundColor: graphSeries[1].fillColor,
            borderColor: graphSeries[1].strokeColor,
            borderWidth: 1,
            order: 2,
          },
          {
            type: 'bar',
            yAxisID: 'y-axis-1',
            label: this.dataLabels[2],
            data: this.chartData[2],
            backgroundColor: graphSeries[2].fillColor,
            borderColor: graphSeries[2].strokeColor,
            borderWidth: 1,
            order: 3,
          },
          {
            type: 'bar',
            yAxisID: 'y-axis-1',
            label: this.dataLabels[3],
            data: this.chartData[3],
            backgroundColor: graphSeries[3].fillColor,
            borderColor: graphSeries[3].strokeColor,
            borderWidth: 1,
            order: 4,
          },
          {
            type: 'line',
            yAxisID: 'y-axis-1',
            label: this.dataLabels[4],
            data: this.chartData[4],
            pointBackgroundColor: 'rgba(0,0,0,0)',
            pointBorderColor: 'rgba(0,0,0,0)',
            borderColor: graphSeries[4].strokeColor,
            borderWidth: 3,
            fill: false,
            order: 0,
            borderDash: [4, 4],
          },
          {
            type: 'line',
            yAxisID: 'y-axis-2',
            label: this.dataLabels[5],
            data: this.chartData[5],
            pointBackgroundColor: 'rgba(0,0,0,0)',
            pointBorderColor: 'rgba(0,0,0,0)',
            borderColor: graphSeries[4].strokeColor,
            borderWidth: 3,
            fill: false,
            order: 0,
            lineTension: 0,
          },
        ],
      }
    },
    tableHeaders() {
      return [
        { text: this.$t('日付'), value: 'text' },
        ...(this.tableLabels as string[]).map((text, i) => {
          return { text, value: String(i), align: 'end' }
        }),
      ]
    },
    tableData() {
      return this.labels
        .map((label, i) => {
          return Object.assign(
            { text: label },
            ...(this.dataLabels as string[]).map((_, j) => {
              if (this.chartData[j][i] === null) {
                return {
                  [j]: '',
                }
              }
              return {
                [j]: this.getFormatter(j)(this.chartData[j][i]),
              }
            })
          )
        })
        .sort((a, b) => dayjs(a.text).unix() - dayjs(b.text).unix())
        .reverse()
    },
    displayOption() {
      const self = this
      const unit = this.unit
      const options: Chart.ChartOptions = {
        tooltips: {
          displayColors: false,
          callbacks: {
            label: (tooltipItem) => {
              const cases = this.getFormatter(tooltipItem.datasetIndex!)(
                parseFloat(tooltipItem.value!)
              )
              let label = `${
                this.dataLabels[tooltipItem.datasetIndex!]
              } : ${cases} ${this.$t('人')}`
              if (tooltipItem.datasetIndex! >= 5) {
                label = `${
                  this.dataLabels[tooltipItem.datasetIndex!]
                } : ${cases} ${unit}`
              }
              return label
            },
            title(tooltipItem, data) {
              const label = data.labels![tooltipItem[0].index!].toString()
              return self.$d(new Date(label), 'dateWithoutYear')
            },
          },
        },
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        scales: {
          xAxes: [
            {
              id: 'day',
              stacked: true,
              gridLines: {
                display: false,
              },
              ticks: {
                fontSize: 9,
                maxTicksLimit: 20,
                fontColor: '#808080',
                maxRotation: 0,
                callback: (label: string) => {
                  return dayjs(label).format('D')
                },
              },
              // #2384: If you set "type" to "time", make sure that the bars at both ends are not hidden.
              // #2384: typeをtimeに設定する時はグラフの両端が見切れないか確認してください
            },
            {
              id: 'month',
              stacked: true,
              gridLines: {
                drawOnChartArea: false,
                drawTicks: true,
                drawBorder: false,
                tickMarkLength: 10,
              },
              ticks: {
                fontSize: 11,
                fontColor: '#808080',
                padding: 3,
                fontStyle: 'bold',
              },
              type: 'time',
              time: {
                unit: 'month',
                displayFormats: {
                  month: 'MMM',
                },
              },
            },
          ],
          yAxes: [
            {
              id: 'y-axis-1',
              position: 'left',
              stacked: true,
              gridLines: {
                display: true,
                drawOnChartArea: true,
                color: '#E5E5E5', // #E5E5E5
              },
              ticks: {
                suggestedMin: 0,
                maxTicksLimit: 8,
                fontColor: '#808080', // #808080
                suggestedMax: this.scaledTicksYAxisMax,
              },
            },
            {
              id: 'y-axis-2',
              position: 'right',
              gridLines: {
                display: true,
                drawOnChartArea: false,
                color: '#E5E5E5', // #E5E5E5
              },
              ticks: {
                suggestedMin: 0,
                maxTicksLimit: 8,
                fontColor: '#808080', // #808080
                suggestedMax: this.scaledTicksYAxisMaxRight,
                callback(value) {
                  return `${value}%`
                },
              },
            },
          ],
        },
      }
      if (this.$route.query.ogp === 'true') {
        Object.assign(options, { animation: { duration: 0 } })
      }
      return options
    },
    displayDataHeader() {
      const sums = Array.from(this.displayData.datasets[0].data.keys()).map(
        (i) =>
          this.displayData.datasets[0].data[i] +
          this.displayData.datasets[1].data[i] +
          this.displayData.datasets[2].data[i] +
          this.displayData.datasets[3].data[i]
      )
      const max = sums.reduce((a, b) => Math.max(a, b), 0)
      const n = sums.indexOf(max)
      return {
        labels: ['2020-01-01'],
        datasets: [
          {
            data: [this.displayData.datasets[0].data[n]],
            backgroundColor: 'transparent',
            yAxisID: 'y-axis-1',
            borderWidth: 0,
          },
          {
            data: [this.displayData.datasets[1].data[n]],
            backgroundColor: 'transparent',
            yAxisID: 'y-axis-1',
            borderWidth: 0,
          },
          {
            data: [this.displayData.datasets[2].data[n]],
            backgroundColor: 'transparent',
            yAxisID: 'y-axis-1',
            borderWidth: 0,
          },
          {
            data: [this.displayData.datasets[3].data[n]],
            backgroundColor: 'transparent',
            yAxisID: 'y-axis-1',
            borderWidth: 0,
          },
          {
            data: [0],
            backgroundColor: 'transparent',
            yAxisID: 'y-axis-1',
            borderWidth: 0,
          },
          {
            data: [this.displayData.datasets[5].data[n]],
            backgroundColor: 'transparent',
            yAxisID: 'y-axis-2',
            borderWidth: 0,
          },
        ],
      }
    },
    displayOptionHeader() {
      const options: Chart.ChartOptions = {
        maintainAspectRatio: false,
        legend: {
          display: false,
        },
        tooltips: { enabled: false },
        scales: {
          xAxes: [
            {
              id: 'day',
              stacked: true,
              gridLines: {
                display: false,
              },
              ticks: {
                fontSize: 9,
                maxTicksLimit: 20,
                fontColor: 'transparent',
                maxRotation: 0,
                minRotation: 0,
                callback: (label: string) => {
                  return dayjs(label).format('D')
                },
              },
            },
            {
              id: 'month',
              stacked: true,
              gridLines: {
                drawOnChartArea: false,
                drawTicks: false, // true -> false
                drawBorder: false,
                tickMarkLength: 10,
              },
              ticks: {
                fontSize: 11,
                fontColor: 'transparent', // #808080
                padding: 13, // 3 + 10(tickMarkLength)
                fontStyle: 'bold',
              },
              type: 'time',
              time: {
                unit: 'month',
              },
            },
          ],
          yAxes: [
            {
              id: 'y-axis-1',
              type: 'linear',
              position: 'left',
              stacked: true,
              gridLines: {
                display: true,
                drawOnChartArea: false,
                color: '#E5E5E5', // #E5E5E5
              },
              ticks: {
                suggestedMin: 0,
                maxTicksLimit: 8,
                fontColor: '#808080', // #808080
                suggestedMax: this.scaledTicksYAxisMax,
              },
            },
            {
              id: 'y-axis-2',
              type: 'linear',
              position: 'right',
              stacked: true,
              gridLines: {
                display: true,
                drawOnChartArea: false,
                color: '#E5E5E5', // #E5E5E5
              },
              ticks: {
                suggestedMin: 0,
                maxTicksLimit: 8,
                fontColor: '#808080', // #808080
                suggestedMax: this.scaledTicksYAxisMaxRight,
                callback(value) {
                  return `${value}%`
                },
              },
            },
          ],
        },
        animation: { duration: 0 },
      }
      return options
    },
    scaledTicksYAxisMax() {
      return Array.from(this.chartData[0].keys())
        .map(
          (i) =>
            this.chartData[0][i] +
            this.chartData[1][i] +
            this.chartData[2][i] +
            this.chartData[3][i]
        )
        .reduce((a, b) => Math.max(a, b), 0)
    },
    scaledTicksYAxisMaxRight() {
      return this.chartData[5].reduce((a, b) => Math.max(a, b), 0)
    },
  },
  methods: {
    onClickLegend(i) {
      this.displayLegends[i] = !this.displayLegends[i]
      this.displayLegends = this.displayLegends.slice()
    },
  },
  mounted() {
    const barChart = this.$refs.barChart as Vue
    const barElement = barChart.$el
    const canvas = barElement.querySelector('canvas')
    const labelledbyId = `${this.titleId}-graph`

    if (canvas) {
      canvas.setAttribute('role', 'img')
      canvas.setAttribute('aria-labelledby', labelledbyId)
    }
  },
}

export default Vue.extend(options)
</script>

<style module lang="scss">
.Graph {
  &Legend {
    text-align: center;
    list-style: none;
    padding: 0 !important;
    li {
      display: inline-block;
      margin: 0 3px;
      div {
        height: 12px;
        margin: 2px 4px;
        width: 40px;
        display: inline-block;
        vertical-align: middle;
        border-width: 1px;
        border-style: solid;
      }
      button {
        color: $gray-3;
        @include font-size(12);
      }
    }
  }
}
</style>
