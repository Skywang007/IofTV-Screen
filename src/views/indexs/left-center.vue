<template>
  <Echart id="leftCenter" :options="options" class="left_center_inner" v-if="pageflag" ref="charts" />
  <Reacquire v-else @onclick="getData" style="line-height:200px">
    重新获取
  </Reacquire>
</template>

<script>
import { currentGET } from 'api/modules'
import { POST, GET } from "@/api/api";
export default {
  data () {
    return {
      pieData: {
        total: 0,
        Orthohair: 0,
        OrthohairOrange: 0,
        Parahairs: 0,
        YellowDragon: 0,
        MoneyBelly: 0,
        DenseLeaves: 0,
        FalseWestern: 0,
      },
      options: {},
      countUserNumData: {
        lockNum: 0,
        onlineNum: 0,
        offlineNum: 0,
        totalNum: 0
      },
      pageflag: true,
      timer: null
    };
  },
  created () {
    this.getData()
  },
  mounted () {
  },
  beforeDestroy () {
    this.clearData()

  },
  methods: {
    clearData () {
      if (this.timer) {
        clearInterval(this.timer)
        this.timer = null
      }
    },
    async getData () {
      this.pageflag = true
      // this.pageflag =false
      const { data, state } = await GET("/api/v1/getBreed", {})
      console.log('initdata', data);
      if (state == 200) {
        this.pieData = data[0]
        this.$nextTick(() => {
            this.init()
          })
      } else {
        this.pageflag = false
        this.$Message({
          text: res.msg,
          type: 'warning'
        })
      }

      // currentGET('big1').then(res => {
      //   //只打印一次
      //   if (!this.timer) {
      //     console.log("设备总览", res);
      //   }
      //   if (res.success) {
      //     this.countUserNumData = res.data
      //     this.$nextTick(() => {
      //       this.init()
      //     })

      //   } else {
      //     this.pageflag = false
      //     this.$Message({
      //       text: res.msg,
      //       type: 'warning'
      //     })
      //   }
      // })
    },
    //轮询
    switper () {
      if (this.timer) {
        return
      }
      let looper = (a) => {
        this.getData()
      };
      this.timer = setInterval(looper, this.$store.state.setting.echartsAutoTime);
      let myChart = this.$refs.charts.chart
      myChart.on('mouseover', params => {
        this.clearData()
      });
      myChart.on('mouseout', params => {
        this.timer = setInterval(looper, this.$store.state.setting.echartsAutoTime);
      });
    },
    init () {
      let total = this.pieData.total;
      let colors = ["#ECA444", "#33A1DB", "#56B557",'4FADEE','#F6B273',"#78D96A",'red'];
      let piedata = {
        name: "种植品种规模分析",
        type: "pie",
        radius: ["42%", "65%"],
        avoidLabelOverlap: false,
        itemStyle: {
          borderRadius: 4,
          borderColor: "rgba(0,0,0,0)",
          borderWidth: 2,
        },

        color: colors,
        data: [
          {
            value: this.pieData.Orthohair,
            name: "正毛",
            label: {
              shadowColor: colors[0],
            },
          },
          {
            value: this.pieData.Parahairs,
            name: "副毛",
            label: {
              shadowColor: colors[2],
            },
          },
          {
            value: this.pieData.OrthohairOrange,
            name: "正毛橘红",
            label: {
              shadowColor: colors[1],
            },
          },
          {
            value: this.pieData.YellowDragon,
            name: "黄龙",
            label: {
              shadowColor: colors[3],
            },
          },
          {
            value: this.pieData.MoneyBelly,
            name: "金钱肚",
            label: {
              shadowColor: colors[4],
            },
          },
          {
            value: this.pieData.DenseLeaves,
            name: "密叶",
            label: {
              shadowColor: colors[5],
            },
          },
          {
            value: this.pieData.FalseWestern,
            name: "假西洋",
            label: {
              shadowColor: colors[6],
            },
          },


        ],
      };
      this.options = {
        title: {
          // zlevel: 0,
          text: ["{value|" + total + "}", "{name|总数}"].join("\n"),
          top: "center",
          left: "center",
          textStyle: {
            rich: {
              value: {
                color: "#ffffff",
                fontSize: 24,
                fontWeight: "bold",
                lineHeight: 20,
              },
              name: {
                color: "#ffffff",
                lineHeight: 20,
              },
            },
          },
        },
        tooltip: {
          trigger: "item",
          backgroundColor: "rgba(0,0,0,.6)",
          borderColor: "rgba(147, 235, 248, .8)",
          textStyle: {
            color: "#FFF",
          },
        },
        legend: {
          show: false,
          top: "5%",
          left: "center",
        },
        series: [
          //展示圆点
          {
            ...piedata,
            tooltip: { show: true },
            label: {
              formatter: "   {b|{b}}   \n   {c|{c}个}   {per|{d}%}  ",
              //   position: "outside",
              rich: {
                b: {
                  color: "#fff",
                  fontSize: 12,
                  lineHeight: 26,
                },
                c: {
                  color: "#31ABE3",
                  fontSize: 14,
                },
                per: {
                  color: "#31ABE3",
                  fontSize: 14,
                },
              },
            },
            labelLine: {
              length: 20, // 第一段线 长度
              length2: 36, // 第二段线 长度
              show: true,

            },
            emphasis: {
              show: true,
            },
          },
          {
            ...piedata,
            tooltip: { show: true },
            itemStyle: {},
            label: {
              backgroundColor: "inherit", //圆点颜色，auto：映射的系列色
              height: 0,
              width: 0,
              lineHeight: 0,
              borderRadius: 2.5,
              shadowBlur: 8,
              shadowColor: "auto",
              padding: [2.5, -2.5, 2.5, -2.5],
            },
            labelLine: {
              length: 20, // 第一段线 长度
              length2: 36, // 第二段线 长度
              show: false,
            },
          },
        ],
      };
    },
  },
};
</script>
<style lang='scss' scoped>
</style>