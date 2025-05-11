<script>
import {mapActions} from 'vuex'

export default {
  name: "MapComponent",
  data() {
    return {
      center: {lat: 39.91799, lng: 116.397027},
      zoom: 10,
      control: {
        scale: {},
        zoom: {
          position: 'topRight',
        },
      },
      geometries: [
        {styleId: 'marker', position: {lat: 39.91799, lng: 116.397027}},
      ],
      styles: {
        marker: {
          width: 20,
          height: 30,
          anchor: {x: 10, y: 30},
        },
      },
      options: {
        minZoom: 5,
        maxZoom: 15,
      },
    }
  },
  computed: {},
  methods: {
    ...mapActions({
      // 映射 this.setBkzhImageAction 为派发 action 的方法
      setImage: 'setBkzhImageAction'
    }),
    initMap() {
      //定义地图中心点坐标
      const center = new TMap.LatLng(19.099419, 109.681782)
      //定义map变量，调用 TMap.Map() 构造函数创建地图
      const map = new TMap.Map(document.getElementById('tmap'), {
        baseMap: {
          // SatelliteBaseMap 对象规范 https://lbs.qq.com/webApi/javascriptGL/glDoc/docIndexMap#6
          type: 'satellite',
          features: ['base', 'road'],
        },
        center: center,//设置地图中心点坐标
        zoom: 8,   //设置地图缩放级别
        pitch: 0,  //设置俯仰角
        rotation: 0,    //设置地图旋转角度
      });
      const marker = new TMap.MultiMarker({
        id: "marker-layer", //图层id
        map: map,
        styles: { //点标注的相关样式
          "marker": new TMap.MarkerStyle({
            "width": 25,
            "height": 35,
            "anchor": {x: 16, y: 32},
            "src": "img/marker.png"
          })
        },
        geometries: null,
      });
      this.$shared.map = map
      this.$shared.marker = marker

      // 2. 监听地图点击事件
      marker.on('hover', function (event) {
        const base64 = (
            event &&
            event.geometry &&
            event.geometry.properties &&
            event.geometry.properties.picture
        ) || ''
        if (!base64) return
        this.setImage(base64)
      });
    }
  },
  mounted() {
    //加载地图
    this.initMap();
  }
}
</script>

<template>
  <div style="flex: 1">
    <div id="tmap"></div>
  </div>
</template>

<style scoped>

</style>