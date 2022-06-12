<template>
  <v-chart style="width: 100%" :options="option"/>
</template>

<script>
import ECharts from 'vue-echarts'
import "echarts/lib/chart/bar";
import "echarts/lib/chart/line";
import "echarts/lib/chart/pie";
import "echarts/lib/component/tooltip";
import "echarts/lib/component/legend";
import "echarts/lib/component/markPoint";
import "echarts/lib/component/markLine";
import "echarts/lib/component/graphic";
export default {
  name: "scenic-statist",
  components: {
    "v-chart": ECharts
  },
  data() {
    return {
      option: {
        color: ['#3398DB'],
        title: {
          text: '景区区域分析',
          left: 'left'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {            // 坐标轴指示器，坐标轴触发有效
            type: 'shadow'        // 默认为直线，可选为：'line' | 'shadow'
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            type: 'category',
            data: [],
            axisLabel:{
              interval:0,//横轴信息全部显示
            },
            axisTick: {
              alignWithLabel: true
            }
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ],
        series: [
          {
            name: '统计数量',
            type: 'bar',
            barWidth: '60%',
            data: []
          }
        ]
      }
    }
  },
  mounted() {
    this.getScenicStatist();
  },
  methods: {
    getScenicStatist() {
      this.$get('/cos/scenic-info/getScenicByArea').then((r) => {
        r.data.data.forEach(item => {
          this.option.xAxis[0].data.push(item.area);
          this.option.series[0].data.push(item.scenicNum);
        })
      })
    }
  },
}
</script>

<style scoped>

</style>
