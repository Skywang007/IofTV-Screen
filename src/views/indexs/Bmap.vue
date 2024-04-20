<template>
  <div>
    <div id="map" style="height: 810px;"></div>
		<div id="heatmap-buttons">
        <button @click="()=>showHeatmap(1)">0 - 20 亩</button>
        <button @click="()=>showHeatmap(2)">20 - 30 亩</button>
        <button @click="()=>showHeatmap(3)">30 - 40 亩</button>
        <button @click="()=>showHeatmap(4)">40 - 100 亩</button>
        <button @click="()=>showHeatmap(5)">100以上</button>
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
			type: 1,
			color: 'red',
      map: '',
      flag: 0,
    }
  },
  created () {
    this.$nextTick(() => {
      this.initMap()
    })
  },
  mounted () {
    // this.initMap()
  },
  methods: {
		async initMap () {
			const { data } = await GET("/api/v1/querySpace", { type:this.type });
			data.forEach(item => {
				item.name = `${item.name}-${item.space}亩`
			})
			this.mapData = data;

      this.map = new BMap.Map("map");
      this.map.centerAndZoom(new BMap.Point(110.628092,21.647925), 11);
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
          padding: "5px"
        });
				marker.setLabel(label);
				
				console.log(this.map.getPanes(), 'abc');
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
</style>