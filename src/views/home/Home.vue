<template>
  <div id="home" class="wrapper">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    <tab-control
      :titles="['流行', '新款', '精选']"
      class="tab-control"
      @tabClick="tabClick"
      ref="tabControl1"
      v-show="isShowBackTop"
    />
    <scroll
      class="content"
      ref="scroll"
      v-bind:probe-type="3"
      @scroll="contentScroll"
      :pull-up-load="true"
      @pullingUp="loadMore"
    >
      <home-swiper :banners="banner" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <recommend-view :recommends="recommend"></recommend-view>
      <feature-view />
      <tab-control
        :titles="['流行', '新款', '精选']"
        class="tab-control"
        @tabClick="tabClick"
        ref="tabControl2"
      />
      <goods-list :goods="showGoods" />
    </scroll>
    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
import HomeSwiper from "./childComps/HomeSwiper";
import RecommendView from "./childComps/RecommendView";
import FeatureView from "./childComps/FeatureView";

import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";
import GoodsList from "components/content/goods/GoodsList";
import Scroll from "components/common/scroll/Scroll";
import BackTop from "components/content/backTop/BackTop";

import { getHomeMultidata, getHomeGoods } from "network/home.js";

export default {
  name: "Home",
  components: {
    HomeSwiper,
    RecommendView,
    FeatureView,
    NavBar,
    TabControl,
    GoodsList,
    Scroll,
    BackTop
  },
  data() {
    return {
      banner: [],
      recommend: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] }
      },
      currentType: "pop",
      isShowBackTop: false,
      saveY: 0
    };
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list;
    }
  },
  created() {
    // 1.请求多个数据
    this.getHomeMultidata();

    // 2.请求商品数据
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  mounted() {
    // 1.图片加载完成的事件监听
    const refresh = this.debounce(this.$refs.scroll.refresh);
    this.$bus.$on("itemImageLoad", () => {
      // console.log("---------");
      // this.$refs.scroll.refresh();
      refresh();
    });
    // 2. 获取TabControl的offsetTop
    // 所有的组件都有$el，用于获取组件中的元素
    // this.tabOffsetTop = this.$refs.TabControl;
  },
  destroyed() {
    // console.log("Home destroyed");
  },
  activated() {
    // console.log("activated");
    this.$refs.scroll.scrollTo(0, this.saveY);
    this.$refs.scroll.refresh();
    // console.log(this.saveY);
  },
  deactivated() {
    // console.log("deactivated");
    this.saveY = this.$refs.scroll.getScrollY();
  },
  methods: {
    // 防抖
    debounce(func, delay = 300) {
      let timer = null;

      return function(...args) {
        if (timer) clearTimeout(timer);

        timer = setTimeout(() => {
          func.apply(this, args);
          // console.log("refresh");
        }, delay);
      };
    },
    // 事件监听相关方法
    tabClick(index) {
      // console.log(index);
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
          break;
      }
      this.$refs.tabControl1.currentIndex = index;
      this.$refs.tabControl2.currentIndex = index;
    },
    backClick() {
      // console.log("返回首页");
      this.$refs.scroll.scroll.scrollTo(0, 0, 500);
    },
    contentScroll(position) {
      // console.log(position);
      this.isShowBackTop = -position.y > 1000;
    },
    loadMore() {
      console.log("上拉加载更多");
      this.getHomeGoods(this.currentType);
    },
    swiperImageLoad() {
      // console.log(this.$refs.tabControl2.$el.offsetTop);
    },
    // 网络请求相关方法
    getHomeMultidata() {
      getHomeMultidata().then(res => {
        // console.log(res);
        this.banner = res.data.banner.list;
        this.recommend = res.data.recommend.list;
      });
    },
    getHomeGoods(type) {
      const page = this.goods[type].page + 1;
      getHomeGoods(type, page).then(res => {
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;
        // console.log(res);
        this.$refs.scroll.finishPullUp();
      });
    }
  }
};
</script>

<style scoped>
#home {
  /*padding-top: 44px;*/
  height: 100vh;
  position: relative;
}

.home-nav {
  background-color: var(--color-tint);
  color: #fff;

  /* position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index: 9; */
}

.tab-control {
  position: relative;
  z-index: 9;
}

.content {
  overflow: hidden;

  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
  right: 0;
}

/*.content {*/
/*height: calc(100% - 93px);*/
/*overflow: hidden;*/
/*margin-top: 44px;*/
/*}*/
</style>
