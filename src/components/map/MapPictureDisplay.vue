<script>
import {mapGetters} from 'vuex'
export default {
  name: 'MapPictureDisplay',
  data() {
    return {
      base64: '',
    }
  },
  methods: {
    // 处理接收到的图片数据

  },
  computed: {
    ...mapGetters(['bkzhImage'])
  },
  watch: {
    // 监听 bkzhImage 的变化
    bkzhImage(newVal, oldVal) {
      console.log('背景图片从', oldVal, '变成了', newVal)
      // 在这里你可以做更多事情，比如：懒加载、动画切换等
      this.base64 = newVal
    }
  },
}
</script>

<template>
  <div>
    <el-image style="width: 100px; height: 100px" :src="base64" fit="cover"></el-image>
  </div>
</template>

<style scoped>

</style>