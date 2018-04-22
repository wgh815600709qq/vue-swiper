<template>
  <div>
    <carousel :imgList="list" :imgHeight="200"  :imgWidth="200" ></carousel>
  </div>
</template>

<script>
import carousel from './carousel.vue'
export default {
  name: 'test',
  data () {
    return {
      list: []
    }
  },
  mounted () {
    setTimeout(() => {
     this.list = [{url: require('../assets/logo1.png')},{url: require('../assets/logo2.png')},{url: require('../assets/logo3.png')}]
    }, 2000)
  },
  components: {
    carousel
  }
}
</script>

<style lang="css" scoped>
</style>