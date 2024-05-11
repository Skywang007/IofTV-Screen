<template>
  <Echart
    id="rightTop"
    :options="option"
    class="right_top_inner"
    v-if="pageflag"
    ref="charts"
  />
  <Reacquire v-else @onclick="getData" style="line-height: 200px">
    重新获取
  </Reacquire>
</template>

<script>
import { currentGET } from "api/modules";
import { graphic } from "echarts";
import { formatTime } from "@/utils/index";
import { POST, GET } from "@/api/api";
export default {
  data() {
    return {
      option: {},
      year: [],
      total: [],
      total2: [],
      pageflag: false,
      timer: null,
    };
  },
  created() {},

  mounted() {
    this.getData();
  },
  beforeDestroy() {
    this.clearData();
  },
  methods: {
    clearData() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = null;
      }
    },
    async getData() {
      this.pageflag = true;
      // this.pageflag =false

      const { data, state, data2 } = await GET("/api/v1/getHistory", {});
      if (state == 200) {
        this.year = data.map((item) => {
          return item.year;
        });
        this.total = data.map((item) => {
          return item.total;
        });
        this.total2 = data2.map((item) => {
          return item.total;
        });
      }

      currentGET("big4").then((res) => {
        if (!this.timer) {
          console.log("报警次数", res);
        }
        if (res.success) {
          this.countUserNumData = res.data;
          this.$nextTick(() => {
            this.init(res.data.dateList, res.data.numList, res.data.numList2);
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
    switper() {
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
    init(xData, yData, yData2) {
      this.option = {
        xAxis: {
          type: "category",
          data: this.year,
          boundaryGap: false, // 不留白，从原点开始
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
            name: "种植面积/亩",
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
            },
          },
          {
            name: "产量",
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
            data: this.total,
            type: "line",
            smooth: true,
            symbol: "circle", //去除点
            name: "种植面积/亩",
            color: "rgba(252,144,16,.7)",
            areaStyle: {
              //右，下，左，上
              color: new graphic.LinearGradient(0,0,0,1,
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
            label: {
              show: true,
              position: "top",
              color: "rgba(252,144,16,.7)",
              formatter: (params) => {
                return `${params.value}亩`;
              },
            },
          },
          {
            data: this.total2,
            type: "line",
            smooth: true,
            yAxisIndex: 1,
            symbol: "circle", //去除点
            name: "产量/吨",
            color: "rgba(0,144,16,.7)",
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
                    color: "rgba(0,144,16,.7)",
                  },
                  {
                    offset: 1,
                    color: "rgba(0,144,16,.0)",
                  },
                ],
                false
              ),
            },
            label: {
              show: true,
              position: "top",
              color: "rgba(0,144,16,.7)",
              formatter: (params) => {
                return `${params.value}吨`;
              },
            },
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