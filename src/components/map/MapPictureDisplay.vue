<script>
import {eventBus} from "@/assets/js/bus";

export default {
  name: 'MapPictureDisplay',
  data() {
    return {
      base64: '',
    }
  },
  created() {
    const that = this
    eventBus.$on('bkzh-image-change', data => {
      if (data && data.base64) {
        that.base64 = data.base64
        eventBus.$emit('open-drawer', {})
      }
    })
  },
}
</script>

<template>
  <div>
    <div style="text-align: left" v-show="base64">
      <div class="title">图片预览</div>
      <el-image :src="base64" fit="cover" class="img"></el-image>
    </div>
  </div>
</template>

<style scoped>
.title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 10px;
}

.img {
  border-radius: 8px;
  border: 1px solid #D7D8D9;
}
</style>