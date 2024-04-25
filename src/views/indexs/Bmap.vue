<template>
  <div>
    <div id="map" style="height: 900px;marginTop:14px"></div>
    <div class="heatmap-buttons">
      <button @click="()=>showHeatmap(1)" class="btn">0 - 20 亩</button>
      <button @click="()=>showHeatmap(2)" class="btn">20 - 30 亩</button>
      <button @click="()=>showHeatmap(3)" class="btn">30 - 40 亩</button>
      <button @click="()=>showHeatmap(4)" class="btn">40 - 100 亩</button>
      <button @click="()=>showHeatmap(5)" class="btn">100以上</button>
    </div>
    <!-- <div id="legend" class="legend" v-if="flag === 0">
      <div class="legend-item" v-for="item in data_legend" :key="item.name">
        <div class="color-box" :style="{ backgroundColor: item.color }"></div>
        <span>{{ item.name }}</span>
      </div>
    </div> -->
  </div>

</template>

<script>
// 引入百度地图
import BMap from 'BMap'
import { GETNOBASE } from "api";
import { POST, GET } from "@/api/api";
export default {
  data () {
    return {
      mapData: [],
      type: 5,
      color: 'orange',
      map: '',
      flag: 0,
    }
  },
  created () {
    this.$nextTick(() => {
      this.initMap()
      this.getBoundary()
    })
  },
  mounted () {
    // this.initMap()
  },
  methods: {
    async initMap () {
      const { data } = await GET("/api/v1/querySpace", { type: this.type });
      data.forEach(item => {
        item.name = `${item.name}-${item.space}亩`
      })
      this.mapData = data;

      this.map = new BMap.Map("map");
      this.map.centerAndZoom(new BMap.Point(110.628092, 21.647925), 11);
      this.map.enableScrollWheelZoom();


      this.mapData.forEach((item) => {

        let lat = parseFloat(item.lat);
        let lng = parseFloat(item.lng);
        let point = new BMap.Point(lng, lat);
        // 创建标注样式
        const markerOptions = {
          icon: new BMap.Symbol(BMap_Symbol_SHAPE_POINT, {
            scale: 1,
            fillColor: this.color, // 使用不同颜色的标记
            fillOpacity: 0.8
          })
        };
        let marker = new BMap.Marker(point, markerOptions);
        // 创建文本标签
        let label = new BMap.Label(item.name, { offset: new BMap.Size(20, -10) });
        label.setStyle({
          color: "black",
          backgroundColor: "white",
          border: "1px solid #ccc",
          padding: "5px",
          fontSize: "18px"
        });
        marker.setLabel(label);

        this.map.getPanes().markerMouseTarget.style.zIndex = 450;//修改图层z-index
        this.map.getPanes().labelPane.style.zIndex = 1000;//修改图层z-index
        // 先隐藏标签
        label.hide();
        // 鼠标悬停时显示工厂名称
        marker.addEventListener("mouseover", function () {
          label.show();
        });

        // 鼠标移开时隐藏工厂名称
        marker.addEventListener("mouseout", function () {
          label.hide();
        });
        // 将标记对象添加到地图上
        this.map.addOverlay(marker);
      })

    },
    getBoundary () {
      var bdary = new BMap.Boundary();
      bdary.get("广东省茂名市化州市", (rs) => {       //获取行政区域
        // this.map.clearOverlays();        //清除地图覆盖物       
        var count = rs.boundaries.length; //行政区域的点有多少个
        if (count === 0) {
          alert('未能获取当前输入行政区域');
          return;
        }
        var EN_JW = "180, 90;";//东北角
        var NW_JW = "-180,  90;";//西北角
        var WS_JW = "-180, -90;";//西南角
        var SE_JW = "180, -90;";//东南角
        //添加环形遮罩层
        var ply1 = new BMap.Polygon(rs.boundaries[0] + SE_JW + SE_JW + WS_JW + NW_JW + EN_JW + SE_JW, {
          strokeColor: "none",
          fillColor: "rgba(0,0,0,0.5)",
          fillOpacity: 1,
          strokeOpacity: 0.5
        }); //建立多边形覆盖物
        this.map.addOverlay(ply1);
        var pointArray = [];
        for (var i = 0; i < count; i++) {
          var ply = new BMap.Polygon(rs.boundaries[i], { strokeWeight: 1, strokeColor: "#ff0000", fillColor: "" }); //建立多边形覆盖物
          this.map.addOverlay(ply);  //添加覆盖物
          pointArray = pointArray.concat(ply.getPath());
        }
        this.map.setViewport(pointArray);
      });
    },
    showHeatmap (index) {
      this.type = index
      switch (index) {
        case 1:
          this.color = 'red'
          break;
        case 2:
          this.color = 'blue'
          break;
        case 3:
          this.color = 'green'
          break;
        case 4:
          this.color = 'yellow'
          break;
        case 5:
          this.color = 'orange'
          break;

        default:
          break;
      }
      this.initMap()
      this.getBoundary()
    }
  },
}
</script>

<style scoped>
.legend {
  display: flex;
  flex-direction: row;
  width: 100%;
  height: 5%;
}

.legend-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  color: white;
}

.color-box {
  width: 15px;
  height: 15px;
  margin-right: 5px;
}
.heatmap-buttons{
  margin-top: 20px;
  margin-left: 10px;
}
.btn{
  margin-right: 10px;
  width: 90px;
  height: 30px;
  background: lightblue;
  color: #4f4d4d;
  border-radius: 8px;
}
</style>