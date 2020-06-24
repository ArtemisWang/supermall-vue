<template>
  <div id="detail">
    <detail-nav-bar />
    <detail-swiper :top-images="topImages" />
  </div>
</template>

<script>
import DetailNavBar from "./childComps/DetailNavBar";
import { getDetail } from "network/detail.js";
import DetailSwiper from "./childComps/DetailSwiper";
export default {
  name: "Detail",
  components: {
    DetailNavBar,
    DetailSwiper
  },
  data() {
    return {
      iid: null,
      topImages: []
    };
  },
  created() {
    // 1. 获取iid
    this.iid = this.$route.params.id;
    // console.log(this.$route.params.id);
    // 2. 请求iid对应的数据
    getDetail(this.iid).then(res => {
      // console.log(res);
      this.topImages = res.result.itemInfo.topImages;
    });
  }
};
</script>

<style scoped>
</style>