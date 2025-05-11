<script setup>

import MapComponent from "@/components/map/MapComponent.vue";
import MapSelecting from "@/components/map/MapSelecting.vue";
import MapPictureDisplay from "@/components/map/MapPictureDisplay.vue";
</script>

<template>
  <div>
    <el-container>
      <el-main>
        <map-component/>
      </el-main>
      <el-aside width="30vw">
        <map-selecting/>
        <map-picture-display/>
      </el-aside>
    </el-container>
  </div>
</template>

<style lang="scss">

</style>