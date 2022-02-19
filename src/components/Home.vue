<template>
  <div class="map" id="map">
  </div>
</template>

<script>
import "ol/ol.css";
import Map from "ol/Map";
import OSM from "ol/source/OSM";
import TileLayer from "ol/layer/Tile";
import ImageLayer from "ol/layer/Image";
import View from "ol/View";
import gfsData from '@/assets/gfs.json'
import ColorMaker from "@/assets/colorMaker.js";
import { fromLonLat, get as getProjection } from "ol/proj";
import {ImageStatic, WMTS as WMTS, XYZ} from "ol/source";
import { getTopLeft, getWidth } from "ol/extent";
import WMTSTileGrid from "ol/tilegrid/WMTS";


export default {
  name: "Home",
  data() {
    return {
      map: null,
    };
  },
  methods: {  
    addOSM: function() {
      this.map = new Map({
        target: "map",
        layers: [
          new TileLayer({
            source: new XYZ({
              url: "http://t0.tianditu.com/DataServer?T=vec_c&x={x}&y={y}&l={z}&tk=74aabc459b20503f659ddee58bf71b25",
              projection: "EPSG:4326"
            })
          }),
        ],
        view: new View({
          projection: "EPSG:4326",
          center: [120, 30],
          zoom: 4,
          //限制地图可可显示的范围,超过这个范围地图将无法拖出去
          extent: [-180, -90, 180, 90]
        }),
      });
      console.log(this.map)
    },

    drawGrid: function () {
      let canvas = document.createElement("canvas");
      let ctx = canvas.getContext("2d");
      let { nx, ny } = gfsData[0].header;
      let data_u = gfsData[0].data;
      let data_v = gfsData[1].data;
      let data = data_u.map((item, index)=>{
        return Math.sqrt(item*item + data_v[index]*data_v[index])
      })
      canvas.width = nx;
      canvas.height = ny;
      let colorMaker = new ColorMaker({
        values: [0, 3, 5, 10, 15, 20, 30],
        colors: [
          [98, 113, 184],
          [61, 110, 163],
          [77, 142, 124],
          [162, 135, 64],
          [151, 75, 145],
          [95, 100, 160],
          [91, 136, 161],
        ],
      });
      //  传进数据 只绘制一次
      console.time('fillRect绘制');
      let imgData = ctx.createImageData(nx, ny);
      for (let i = 0; i < ny; i++) {
        for (let j = 0; j < nx; j++) {
          let dataIndex = i * nx + j;
          let curColors = colorMaker.makeColor(data[dataIndex]);
          imgData.data[dataIndex * 4 + 0] = curColors[0];
          imgData.data[dataIndex * 4 + 1] = curColors[1];
          imgData.data[dataIndex * 4 + 2] = curColors[2];
          imgData.data[dataIndex * 4 + 3] = 255;
        }
      }
      ctx.putImageData(imgData, 0, 0);

      // console.log(canvas.toDataURL())
      console.timeEnd('fillRect绘制');
      return canvas.toDataURL()
    },

    drawPicToMap: function () {
      var projection = getProjection("EPSG:4326");
      var projectionExtent = projection.getExtent();

      let extent = [-180, -90, 180, 90]

      let source = new ImageStatic({
        imageExtent: extent,
        url: this.drawGrid(),
        projection: "EPSG:4326",
        
      });
      var imageLayer = new ImageLayer({
        source: source,
        opacity: 0.8,
      });
      // imageLayer.changed();
      this.map.addLayer(imageLayer);
    },  
  },
  mounted() {
    this.addOSM();
    // this.drawGrid()
    this.drawPicToMap()
  },
};
</script>

<style scoped>
.map {
  height: 700px;
  width: 100%;
  border: 2px solid black;
}
</style>
