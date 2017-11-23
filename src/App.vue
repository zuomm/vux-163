<template>
  <div id="app">
    <view-box>
      <!-- view-box 也带了三个插槽 ：
        header(顶部) ：x-header使用 slot="header"
        default(正文) 
        bottom(底部) ：tabbar使用 slot="bottom"
      -->
      
      <!-- header(顶部) ：x-header使用 slot="header" -->
      <x-header 
        class="header"
        slot="header"
        :left-options="{'showBack':false,'backText':'back'}"
      >
        <di@on-item-click="onItemClick"v slot="left">直播</div>
        <div>网易</div><!--中间插槽默认-->
        <div slot="right">搜索</div>
      </x-header>


      <!-- default(正文) -->
      <!-- <div style="height:3000px;background:red;"></div> -->
      <sc :lock-y="true"><!--锁定纵向滚动-->
        <!-- scroller组件最外层必须要包裹一个div -->
        <div class="tab">
          <tab>
            <!--默认选中高亮加上selected属性-->
            <tab-item selected @on-item-click="onItemClick">推荐</tab-item>
            <tab-item>要闻</tab-item>
            <tab-item>科技</tab-item>
            <tab-item>娱乐</tab-item>
            <tab-item>体育</tab-item>
            <tab-item>互联网</tab-item>
          </tab>
        </div>
      </sc>

      <scroller 
        class="my-scroller"
        :on-refresh="refresh"
        :on-infinite="infinite"
        ref="myRef"
        >
        <!-- refresh-text ="loading" 下拉刷新修改文字 -->
        <swiper
          :list="swiperList"
          v-model="swiperIndex"
          :loop="true"
        ><!--
          :list 数据绑定
          :loop="true" 循环无缝轮播
        -->
        </swiper>

        <marquee class="my-marquee" behavior="" direction="up">
          <marquee-item v-for="i in newsList" :key="i">{{i.title}}</marquee-item>
        </marquee>

        <panel
          :list="dataList"
        >
        </panel>
        <panel 
        :list="moreDateList">
        </panel>
      </scroller>
      <!--bottom(底部)  tabbar:底部切换导航组件 -->
      <tabbar slot="bottom" class="bottom">
        <!--tabbar默认插槽只能是主体内容，
            tabbar-item是放在默认的插槽里面作为内容
        -->
        <tabbar-item>
          <img slot="icon" src="./assets/icon_nav_article.png"><!--图片插槽-->
          <span slot="label">首页</span><!--文字插槽-->
        </tabbar-item>
        <tabbar-item show-dot><!--show-dot:红点-->
          <img slot="icon" src="./assets/icon_nav_button.png">
          <span slot="label">推荐</span>
        </tabbar-item>
        <tabbar-item selected link="/component/demo">
          <img slot="icon" src="./assets/icon_nav_cell.png">
          <span slot="label">我的</span>
        </tabbar-item>
        <!-- <tabbar-item badge="2">
          <img slot="icon" src="">
          <span slot="label">News</span>
        </tabbar-item> -->
      </tabbar>
    </view-box>


    <!--
      <router-view></router-view>：
        路由切换组件template插入的地方
      vue-router插件的作用:将每个路径映射到对应的组件，并通过修改路由进行组件间的切换。
    -->
    <!-- <router-view></router-view> -->
  </div>
</template>

<script>
import { ViewBox,XHeader,Tabbar, TabbarItem ,Tab, 
TabItem ,Scroller as sc,Swiper,Marquee,MarqueeItem,Panel } from 'vux'

const refreshKey = ['A', 'B01', 'B02', 'B03', 'B04', 'B05', 'B06', 'B07', 'B08', 'B09', 'B010'];
let key = 0;
let keyValue = 'A';
let start = 0;//上拉开始条数
let end = start + 9;//上拉每页加载10条数据
let infiniteOn = false;//数据是否加载完成

function getRefreshKey(){
  key++;
  if( key == refreshKey.length){
    key = 0;
  }
  keyValue = refreshKey[key];
}
export default {
  name: 'app',
  components: {
    ViewBox,
    XHeader,
    Tabbar,
    TabbarItem,
    Tab, 
    TabItem,
    sc,
    Swiper,
    Marquee,
    MarqueeItem,
    Panel
  },
  data(){
    return{
      swiperList:[],// meta: {
        //   source: '来源信息',
        //   date: '时间',
        //   other: '其他信息'
        // }],
      swiperIndex:0, //初始化显示第几张
      dataList:[],
      newsList:[],
      moreDateList:[]
    }
  },
  created(){ //组件创建时后加载数据
    this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html')
    .then((data)=>{
      // 首页轮播图数据
      this.swiperList = data.focus.filter( item => {
        return item.addata == null;
      }).map( item => {
        /*
          通过map方法构造出数据真实想要的值
          {
            url: 'javascript:',
            img: 'https://static.vux.li/demo/1.jpg',
            title: '送你一朵fua'
          }
        */ 
        return{
          url: item.link,
          img: item.picInfo[0].url,
          title: item.title
        }
      });

      // 滚动新闻列表
      this.dataList = data.live.filter( item => {
        return item.addata == null;
      }).map( item => {
        /*
          通过map方法构造出数据真实想要的值
          {
              src: 'http://somedomain.somdomain/x.jpg',
              fallbackSrc: 'http://placeholder.qiniudn.com/60x60/3cc51f/ffffff',
              title: '标题一',
              desc: '由各种物质组成的巨型球状天体，叫做星球。星球有一定的形状，有自己的运行轨道。',
              url: '/component/cell'
          }
        */ 
        return{
          url: item.link,
          src: item.picInfo[0].url,
          title: item.title,
          desc:item.title
        }
      });
      infiniteOn = true;

      // 新闻滚动
      this.newsList = data.news.filter( item => {
        return item.addata == null;
      }).map( item => {
        return{
          url: item.link,
          title: item.title,
        }
      });
    });
    
  },
  methods:{
    refresh(){ //下拉刷新执行的回调函数
      // console.log(1);
      //下拉数据刷新
      getRefreshKey();
      this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html',{
        miss:'00',
        refresh:keyValue
      }).then(data=>{
        console.log(data);
        // console.log(this.$refs.myRef);

        // 刷新首页滚动新闻列表
        this.dataList = data.list.filter( item => {
          return item.addata == null;
        }).map( item => {
          /*
            通过map方法构造出数据真实想要的值
            {
                src: 'http://somedomain.somdomain/x.jpg',
                fallbackSrc: 'http://placeholder.qiniudn.com/60x60/3cc51f/ffffff',
                title: '标题一',
                desc: '由各种物质组成的巨型球状天体，叫做星球。星球有一定的形状，有自己的运行轨道。',
                url: '/component/cell'
            }
          */ 
          return{
            url: item.link,
            src: item.picInfo[0].url,
            title: item.title,
            desc:item.title
          }
        });

        this.$vux.toast.text(`已经更新了${this.dataList.length}条数据`, 'top');
        //下拉数据已经加载完成
        this.$refs.myRef.finishPullToRefresh();
      });
    
    },
    infinite(){
      if(!infiniteOn){
        //上拉数据已经加载完成
        this.$refs.myRef.finishInfinite();
        return;
      }
      console.log(2);
      //上拉加载更多
      this.$jsonp(
        `http://3g.163.com/touch/jsonp/sy/recommend/${start}-${end}.html`,{
          miss:'00',
          refresh:keyValue
        }).then((data)=>{
          console.log(data); 

          // 刷新首页滚动新闻列表
          let datalist = data.list.filter( item => {
            return item.addata == null && item.picInfo[0];
          }).map( item => {
            /*
              通过map方法构造出数据真实想要的值
              {
                  src: 'http://somedomain.somdomain/x.jpg',
                  fallbackSrc: 'http://placeholder.qiniudn.com/60x60/3cc51f/ffffff',
                  title: '标题一',
                  desc: '由各种物质组成的巨型球状天体，叫做星球。星球有一定的形状，有自己的运行轨道。',
                  url: '/component/cell'
              }
            */ 
            return{
              url: item.link,
              src: item.picInfo[0].url,
              title: item.title,
              desc:item.title
            }
          });
          this.moreDateList = this.moreDateList.concat(datalist);
          start += 10;
          end = start + 9;

          //上拉数据已经加载完成
          this.$refs.myRef.finishInfinite();
      });
    }
  }
}
</script>

<style lang="less">
  @import '~vux/src/styles/reset.less';
  html, body {
    margin:0;
    height: 100%;
    width: 100%;
    overflow-x: hidden;
  }
  #app{
    height:100%;
    .header{
      position: absolute;
      left: 0;
      top:0;
      width:100%;
      z-index: 100;
    }
    .bottom{
      position: absolute;
      left: 0;
      bottom:0;
      width:100%;
      z-index: 100;
    }
    .tab{
      margin-top:46px;
      width:1000px;
    }
    .my-scroller{
      top:90px;
    }
    .loading-layer{
      padding-bottom:90px;
    }
    .my-marquee{
      margin:10px;
    }
    .weui-media-box__hd,.weui-media-box__hd img{
      width:60px;
      height:60px;
    }
  }
</style>
