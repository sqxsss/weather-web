<template>
  <div>
    <div style="width: 95%;height: 50px;margin-bottom: 20px; margin-top: 10px">
      <el-select v-model="yearValue" clearable placeholder='select' style="margin-left: 200px">
        <el-option
          v-for="item in yearOptions"
          :key="item"
          :label="item"
          :value="item">
        </el-option>
      </el-select>
      <el-select v-model="cityValue" filterable clearable multiple collapse-tags placeholder='select'
                 style="margin-left: 200px; margin-right: 200px">
        <el-option
          v-for="item in cityOptions"
          :key="item"
          :label="item"
          :value="item">
        </el-option>
      </el-select>
      <el-button type="primary" icon="el-icon-search" @click="getSeriesData()">Search</el-button>
    </div>
    <div id="barchartTC" style="width: 95%;height: 800px; margin: 0 auto"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts'

let myChart

export default {
  name: 'TemperatureC',
  data () {
    return {
      yearValue: '2019',
      cityValue: ['GAINESVILLE', 'JACKSONVILLE'],
      yearOptions: ['2019', '2020', '2021'],
      cityOptions: [],
      seriesData: []
    }
  },
  created () {
    this.getSeriesData()
    this.getCityList()
  },
  mounted () {
    this.$nextTick(() => {
      this.setTest()
    })
  },
  methods: {
    getCityList () {
      this.$ajax.get('/api/basicData/location/getCityList').then(res => {
        if (res.status === 200) {
          this.cityOptions = res.data
        }
      })
    },
    getSeriesData () {
      this.seriesData = []
      for (let item of this.cityValue) {
        this.$ajax.post('/api/complexQuery/temperatureComp/getMonthlyDataListByCity/' + this.yearValue + '/' + item.replaceAll('%20', ' ')).then(res => {
          if (res.status === 200) {
            this.seriesData.push({
              name: item.replaceAll('%20', ' '),
              type: 'line',
              data: res.data,
              markPoint: {
                data: [
                  { type: 'max', name: 'Max' },
                  { type: 'min', name: 'Min' }
                ]
              },
              markLine: {
                data: [{ type: 'average', name: 'Avg' }]
              }
            })
            this.setTest()
          }
        })
      }
    },
    setTest () {
      if (myChart !== null && myChart !== '' && myChart !== undefined) {
        myChart.dispose()
      }
      let newPromise = new Promise((resolve) => {
        resolve()
      })
      newPromise.then(() => {
        // console.log('333')
        myChart = echarts.init(document.getElementById('barchartTC'))
        myChart.setOption(option, true)
      })
      let option = {
        title: {
          text: 'Temperature Comparison'
        },
        tooltip: {
          trigger: 'axis'
        },
        toolbox: {
          show: true,
          feature: {
            magicType: { show: true, type: ['line', 'bar'] },
            saveAsImage: { show: true }
          }
        },
        legend: {},
        xAxis: {
          data: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
          type: 'category',
          axisPointer: {
            type: 'shadow'
          }
        },
        yAxis: {
          type: 'value'
        },
        series: this.seriesData
      }
    }
  }
}
</script>

<style scoped>

</style>
