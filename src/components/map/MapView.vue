<script>

import MapComponent from "@/components/map/MapComponent.vue";
import MapSelecting from "@/components/map/MapSelecting.vue";
import MapPictureDisplay from "@/components/map/MapPictureDisplay.vue";
import {eventBus} from "@/assets/js/bus";

export default {
  name: 'MapView',
  components: {
    MapComponent,
    MapSelecting,
    MapPictureDisplay
  },
  data() {
    return {
      drawer: true,
      direction: 'rtl'
    }
  },
  methods: {
    handleClose(done) {
      done();
    }
  },
  created() {
    const that = this
    eventBus.$on('open-drawer', () => {
      console.log("监听到打开侧边栏事件")
      that.drawer = true
    })
  }
}
</script>

<template>
  <div>
    <map-component/>
    <el-drawer
        title="槟榔林影像"
        :visible.sync="drawer"
        :direction="direction"
        :before-close="handleClose">
      <div class="drawer-container">
        <map-selecting/>
        <map-picture-display/>
      </div>
    </el-drawer>
    <div class="options">
      <el-button type="success" @click="drawer = true">打开侧边栏</el-button>
    </div>
  </div>
</template>

<style lang="scss">
.drawer-container {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 20px;
}

.options {
  position: absolute;
  bottom: 20px;
  right: 40px;
  z-index: 1000;
}
</style>