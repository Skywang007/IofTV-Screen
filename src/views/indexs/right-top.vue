<template>
  <Echart id="rightTop" :options="option" class="right_top_inner" v-if="pageflag" ref="charts" />
  <Reacquire v-else @onclick="getData" style="line-height: 200px">
    重新获取
  </Reacquire>
</template>

<script>
import { currentGET } from "api/modules";
import { graphic } from "echarts"
import { formatTime } from '@/utils/index'
import { POST, GET } from "@/api/api";
export default {
  data () {
    return {
      data:null,
      data2:null,
      option: {},
      pageflag: false,
      timer: null,
      colors:['#874CCC','#135D66','#003C43','#FB6D48','#FF204E','#2D9596']
    };
  },
  created () {

  },

  mounted () {
    this.getData();
  },
  beforeDestroy () {
    this.clearData();
  },
  methods: {
    clearData () {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
    },
    async getData () {
      this.pageflag = true;
      // this.pageflag =false

      const { data, state,data2 } = await GET('/api/v1/getSciento', {})
      if (state == 200) {
        this.data = data.map(item=> item.lastestData)
        this.data2 = data2.map(item=> item.lastestData)
      }
      currentGET("big4").then((res) => {
        if (!this.timer) {
          console.log("报警次数", res);
        }
        if (res.success) {
          this.countUserNumData = res.data;
          this.$nextTick(() => {
            this.init(res.data.dateList, res.data.numList, res.data.numList2)
            // this.switper();
          });
        } else {
          this.pageflag = false;
          this.$Message({
            text: res.msg,
            type: "warning",
          });
        }
      });
    },
    //轮询
    switper () {
      if (this.timer) {
        return;
      }
      let looper = (a) => {
        this.getData();
      };
      this.timer = setInterval(
        looper,
        this.$store.state.setting.echartsAutoTime
      );
      let myChart = this.$refs.charts.chart;
      myChart.on("mouseover", (params) => {
        this.clearData();
      });
      myChart.on("mouseout", (params) => {
        this.timer = setInterval(
          looper,
          this.$store.state.setting.echartsAutoTime
        );
      });
    },
    init (xData, yData, yData2) {
      const xData2 = ['2时','4时','6时','8时','10时','12时','14时','16时','18时','20时','22时','24时',]
      this.option = {
        xAxis: {
          type: "category",
          data: xData2,
          boundaryGap: true, // 不留白，从原点开始
          splitLine: {
            show: true,
            lineStyle: {
              color: "rgba(31,99,163,.2)",
            },
          },
          axisLine: {
            // show:false,
            lineStyle: {
              color: "rgba(31,99,163,.1)",
            },
          },
          axisLabel: {
            color: "#7EB7FD",
            fontWeight: "500",
          },
        },
        yAxis: [
          {
            name: "温度",
            type: "value",
            nameTextStyle: {
              color: "#7EB7FD",
              fontWeight: "500",
            },
            splitLine: {
              show: true,
              lineStyle: {
                color: "rgba(31,99,163,.2)",
              },
            },
            axisLine: {
              lineStyle: {
                color: "rgba(31,99,163,.1)",
              },
            },
            axisLabel: {
              color: "#7EB7FD",
              fontWeight: "500",
              formatter: '{value} °C'
            },
          },
          {
            name: '含水量',
            nameTextStyle: {
              color: "#7EB7FD",
              fontWeight: "500",
            },
            type: "value",
            axisLabel: {
              color: "#7EB7FD",
              fontWeight: "500",
              formatter: '{value} °%'
            },
          },
        ],
        tooltip: {
          trigger: "axis",
          backgroundColor: "rgba(0,0,0,.6)",
          borderColor: "rgba(147, 235, 248, .8)",
          textStyle: {
            color: "#FFF",
          },
        },
        grid: {
          //布局
          show: true,
          left: "10px",
          right: "30px",
          bottom: "10px",
          top: "28px",
          containLabel: true,
          borderColor: "#1F63A3",
        },
        series: [
          {
            data: this.data,
            type: "line",
            smooth: true,
            symbol: "none", //去除点
            name: "土壤温度",
            color: "rgba(252,144,16,.7)",
            areaStyle: {
              //右，下，左，上
              color: new graphic.LinearGradient(
                0,
                0,
                0,
                1,
                [
                  {
                    offset: 0,
                    color: "rgba(252,144,16,.7)",
                  },
                  {
                    offset: 1,
                    color: "rgba(252,144,16,.0)",
                  },
                ],
                false
              ),
            },
          },
          {
            data: this.data2,
            type: "bar",
            yAxisIndex:1,
            smooth: true,
            symbol: "none", //去除点
            name: "土壤温度",
            color: "rgba(9,202,243,.7)",
          },
        ],
      };
    },
  },
};
</script>
<style lang='scss' scoped>
.right_top_inner {
  margin-top: -8px;
}
</style>