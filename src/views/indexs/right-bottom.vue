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
      XData: [],
      YData: [],
      pageflag: false,
      timer: null,
      colors:['#874CCC','#135D66','#003C43','#FB6D48','#FF204E','#2D9596']
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

      const { data, state } = await GET("/api/v1/getCapacity", {});
      this.XData = data.map((item) => {
        return item.county;
      });
      let lastGeneratedIndex;
      this.YData = data.map((item) => {
        let currentColorIndex;
        do {
          currentColorIndex = Math.floor(Math.random() * 6);
        } while (currentColorIndex === lastGeneratedIndex);

        lastGeneratedIndex = currentColorIndex;

        const color = this.colors[currentColorIndex];
        return {
          value: item.total,
          itemStyle: {
            color: color,
          },
        };
      });
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
      const xData2 = ["2时", "4时", "6时", "8时", "10时", "12时"];
      this.option = {
        xAxis: {
          type: "category",
          data: this.XData,
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
        yAxis: {
          name: "产能/吨",
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
        tooltip: {
          trigger: "axis",
          backgroundColor: "rgba(0,0,0,.6)",
          borderColor: "rgba(147, 235, 248, .8)",
          textStyle: {
            color: "#FFF",
          },
        },
        // grid: {
        //   //布局
        //   show: true,
        //   left: "10px",
        //   right: "30px",
        //   bottom: "10px",
        //   top: "28px",
        //   containLabel: true,
        //   borderColor: "#1F63A3",
        // },
        series: [
          {
            data: this.YData,
            type: "bar",
            smooth: true,
            symbol: "none", //去除点
            name: "产能/吨",
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
            label: {
              show: true,
              position: "top",
              color: "#7EB7FD",
              formatter: (params) => {
                return `${params.value}吨`;
              },
            },
          },
        ],
        dataZoom: [
          {
            type: "slider",
            xAxisIndex: 0,
            start: 20,
            end: 40,
            show: true,
            // bottom:"-20px",
            // height: 10,
            // handleIcon:
            //   "path://M10.7,11.9v-1.3H9.3v1.3c-4.9,0.3-8.8,4.4-8.8,9.4c0,5,3.9,9.1,8.8,9.4v1.3h1.4v-1.3c4.9-0.3,8.8-4.4,8.8-9.4C19.5,16.3,15.6,12.2,10.7,11.9z M13.3,24.4H6.7V23h6.6V24.4z M13.3,19.6H6.7v-1.4h6.6V19.6z",
            // handleSize: "100%",
            textStyle: {
              color: "#7EB7FD",
            },
            borderColor: "#1F63A3",
            backgroundColor: "rgba(31, 99, 163, 0.05)",
            fillerColor: "rgba(31, 99, 163, 0.15)",
            emphasis: {
              handleStyle: {
                color: "#FC9010",
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