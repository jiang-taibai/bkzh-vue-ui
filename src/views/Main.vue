<template>
  <div>
    <el-container style="height: 100vh">
      <!-- 导航栏 -->
      <el-header style="background-color: rgba(0, 85, 255, 1);">
        <div>
          <el-row type="flex">
            <el-col :span="20" class="hidden-md-and-down">
              <el-menu class="el-menu-traj" mode="horizontal" background-color="rgba(0, 85, 255, 1)"
                       text-color="#fffefd" active-text-color="#ffffff" style="border-bottom: none;">
                <el-row type="flex">
                  <!-- 标题 -->
                  <el-col :span="7">
                    <el-menu-item>
                      <div class="title-span">船舶轨迹大数据可视化系统</div>
                    </el-menu-item>
                  </el-col>
                  <!-- 菜单 -->
                  <!-- 虚拟交通网络 -->
                  <el-col :span="1.5">
                    <el-menu-item>
                      <div class="func-span">AIS数据</div>
                    </el-menu-item>
                  </el-col>
                  <!-- 轨迹聚类 -->
                  <el-col :span="1.5">
                    <el-menu-item>
                      <div class="func-span">卫星图像</div>
                    </el-menu-item>
                  </el-col>
                  <!-- 可视化面板 -->
                  <el-col :span="1.5">
                    <el-menu-item>
                      <div class="func-span" @click="skipToVisual">可视化面板</div>
                    </el-menu-item>
                  </el-col>
                  <!-- 联系我们 -->
                  <el-col :span="1.5">
                    <el-menu-item>
                      <div class="func-span" @click="skipToContact">联系我们</div>
                    </el-menu-item>
                  </el-col>
                  <!-- 更多功能 -->
                  <el-col :span="1.5">
                    <el-menu-item>
                      <el-dropdown @command="skipToMore">
                        <div class="func-span el-dropdown-link">
                          更多功能<i class="el-icon-arrow-down el-icon--right"></i>
                        </div>
                        <el-dropdown-menu slot="dropdown">
                          <el-dropdown-item icon="el-icon-circle-check" command="compress">
                            轨迹压缩
                          </el-dropdown-item>
                        </el-dropdown-menu>
                      </el-dropdown>
                    </el-menu-item>
                  </el-col>
                </el-row>
              </el-menu>
            </el-col>
            <!-- 菜单图标 -->
            <el-col :xs="9" class="hidden-md-and-up">
              <div class="nav-icon" @click="drawer = true">
                <i class="el-icon-s-unfold" style="font-size: 30px;"></i>
              </div>
            </el-col>
            <!-- 登录 -->
            <el-col :span="4" :xs="15">
              <div class="rightsection">
                <div v-if="$store.state.isLogin == true">
                  欢迎您，{{ $store.state.userName }}
                  <el-button type="primary" @click="logout" style="margin-left: 5px;">退出</el-button>
                </div>
                <span class="btn-click" v-else @click="skipToLogin">登录</span>
              </div>
            </el-col>
          </el-row>
        </div>
      </el-header>
      <el-main style="padding: 0em;">
        <map-view class="map-view-container"/>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import 'leaflet-minimap/dist/Control.MiniMap.min.css'
import 'leaflet-fullscreen/dist/leaflet.fullscreen.css'
import SecondaryImage from '../components/SecondaryImage.vue'
import MapView from "@/components/map/MapView.vue";


export default {
  name: "Main",
  components: {
    MapView,
    SecondaryImage
  },
  data() {
    return {
      map: "",     //地图
      trajIcon: "",     //图标
      layerGroup1: null,
      layerGroup: null,    //船舶图层组
      // point: ""
      tianditu_1_tile: "",     //天地图矢量底图
      tianditu_1_marker: "",      //天地图矢量标记
      tianditu_2_tile: "",     //天地图影像底图
      tianditu_2_marker: "",      //天地图影像标记
      tianditu_3_tile: "",     //天地图地形底图
      tianditu_3_marker: "",      //天地图地形标记
      basicLayer: "",       //底图图层
      markerLayer: "",      //注解图层
      loading: true,        //加载动画
      drawer: false,        //抽屉显示
      direction: "ltr",     //抽屉方向
      trajMMSI: "",      //船舶MMSI
      trajSign: "",      //船舶呼号
      trajIMO: "",       //船舶IMO
      trajVesselType: "", //船舶类型
      trajVesselLength: "",   //船舶船长
      trajVesselWidth: "",    //船舶船宽
      trajDeepMax: "",    //船舶吃水
      trajHeading: "",   //船舶船首向
      trajCourse: "",    //船舶船迹向
      trajSpeed: "",     //船舶航速
      trajLat: "",       //船舶纬度
      trajLon: "",       //船舶经度
      trajTarget: "",    //船舶目的地
      trajTimeStamp: "", //船舶更新时间
      trajVesselName: "",     //船舶船名
      countryPic: "",    //国家图片
      txtShow: false,    //搜索框的x是否可见
      contShow: false,   //内容框是否可见
      mapLon: "",     //经度
      mapLat: "",     //纬度
      zoomLevel: "",  //缩放级别
      shipNum: "",    //船舶数量
      centerIcon: "",     //中心点图标
      drawControl: null,    //画图控件
      drawLayerGroup: null,    //图形图层组
      drawObj: null,     //绘制对象
      drawType: null,    //绘制类型
      myChartStyle: {float: "left", width: "100%", height: "340px"},//图表样式
      image_id: [],
      img_list: [], //所有大图的列表
      shipPath: 'ship0.jpg',
      imagenumber: 0,
      imageSet: [{
        url: 'ship1.jpg'
      }, {
        url: 'ship2.jpg'
      }],
      isCollapse: true,//隐藏右侧标记
      isCollapse1: true, //隐藏左侧标记
    };
  },
  //将imageList中获取的数据传到boatList中
  methods: {
    //跳转到登录界面
    skipToLogin() {
      this.$router.push("/login");
    },
    //跳转到可视化面板
    skipToVisual() {
      this.drawer = false;     //关闭抽屉
      this.$router.push("/visualization");
    },
    //跳转到联系我们
    skipToContact() {
      this.drawer = false;     //关闭抽屉
      this.$router.push("/contact");
    },
    //跳转到轨迹压缩
    skipToMore(command) {
      this.drawer = false;     //关闭抽屉
      if (command == "compress") {
        this.$router.push("/more/compress");
      }

    },
    //注销
    logout() {
      localStorage.removeItem("Flag");
      this.$store.dispatch("userLogin", false);
    },
  },
  mounted() {
    // //初始化地图
    // this.initMap();
    // //初始化绘制控件
    // this.initDrawCtrl();
    //登录状态判断
    if (localStorage.getItem("Flag") === 'isLogin') {
      this.$store.state.isLogin = true;
      this.$store.state.userName = localStorage.getItem("userName");
    }
    this.zoomLevel = this.map.getZoom();   //初始化缩放级别
  }
}
</script>

<style lang="scss">

#left, #right {
  width: 50px;
  height: 70px;
  background-color: rgba(0, 0, 255, 0.8);
  position: absolute;
  top: 520px;
  color: #fff;
  text-align: center;
  line-height: 65px;
  font-size: 65px;
  z-index: 99
}

#left {
  left: 0;
}

#right {
  right: 0;
}

.el-header,
.el-footer {
  background-color: #B3C0D1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  color: #333;
  text-align: center;
}

.el-main {
  color: #333;
  text-align: center;
}

.el-container:nth-child(5) .el-aside,
.el-container:nth-child(6) .el-aside {
  line-height: 260px;
}

.el-container:nth-child(7) .el-aside {
  line-height: 320px;
}


* {
  padding: 0;
  margin: 0;
}

.title-span {
  font-size: 25px;
  text-align: left;
  font-weight: bold;
  line-height: calc(7vh);
  overflow: hidden;
}

.func-span {
  text-align: center;
  font-size: 18px;
  line-height: calc(7vh);
  color: #ffffff;
}

.el-menu-item:hover {
  background-color: rgba($color: #ffffff, $alpha: 0.2) !important;
}

.rightsection {
  text-align: right;
  line-height: calc(7vh);
  font-size: 18px;
  color: #ffffff;
}

.btn-click {
  padding: 15px;
}

.btn-click:hover {
  cursor: pointer;
  background-color: rgba($color: #ffffff, $alpha: 0.2);
}

.nav-icon {
  color: #ffffff;
  line-height: calc(8vh);
}

.nav-icon:hover {
  cursor: pointer;
}

#map {
  width: 100%;
  height: calc(93vh);
  z-index: 1;
}

.search_box {
  position: absolute;
  width: 430px;
  line-height: 40px;
  border-radius: 2px;
  font-size: 16px;
  top: 10px;
  left: 20px;
  z-index: 1000;
  background: 0 0;
  background-color: blue;
  overflow: hidden;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.25);
}

.pos_r {
  display: block;
  position: relative;
}

.search_box input {
  border: none;
  background: #fff;
  width: 340px;
  padding-left: 20px;
  padding-right: 10px;
  height: 40px;
  line-height: 40px;
  float: left;
  font-size: 14px;
}

.left_redius {
  border-radius: 2px 0 0 2px;
}

.right_radius {
  border-radius: 0 2px 2px 0;
}

.search_btn {
  display: inline-block;
  width: 60px;
  text-align: center;
  background-image: url("../assets/search_btn.png");
  background-color: #2770d4;
  height: 40px;
  line-height: 30px;
  background-position: center center;
  background-repeat: no-repeat;
  float: right;
}

a {
  color: #333;
  text-decoration: none;
}

a:hover {
  cursor: pointer;
}

.clear_input_btn {
  display: block;
  right: 71px;
  width: 20px;
  height: 20px;
  border-radius: 20px;
  background: #ddd;
  top: 10px;
  // color: #fff;
  line-height: 20px;
  text-align: center;
  font-size: 18px;
  position: absolute;
}

input {
  -webkit-writing-mode: horizontal-tb !important;
  text-rendering: auto;
  color: -internal-light-dark(black, white);
  letter-spacing: normal;
  word-spacing: normal;
  text-transform: none;
  text-indent: 0px;
  text-shadow: none;
  display: inline-block;
  text-align: start;
  appearance: auto;
  -webkit-rtl-ordering: logical;
  cursor: text;
  font: 400 13.3333px arial;
}

.content_box {
  z-index: 5688;
  width: 430px;
  height: 325px;
  top: 60px;
  left: 20px;
  display: block;
  margin: 0;
  padding: 0;
  background-color: #fff;
  -webkit-background-clip: content;
  border-radius: 2px;
  position: relative;
  pointer-events: auto;
  box-shadow: 0 2px 2px rgba(0, 0, 0, .25) !important;
}

.content_title {
  padding: 0;
  border: 0;
  background-color: #2770d4;
  border-radius: 0;
  line-height: 26px;
  height: 40px;
  font-size: 14px;
  color: #333;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.content_title span img {
  width: 50px;
  height: 26px;
  margin-top: 7px;
  margin-left: 15px;
  float: left;
}

.content_time_span:hover {
  color: blue;
  cursor: pointer;
  text-decoration: underline;
}

.vessel_name {
  margin-left: 10px;
  margin-top: 8px;
  color: #fff;
  float: left;
  font-size: 16px;
}

.close_btn {
  margin-top: 5px;
  margin-right: 10px;
  float: right;
}

.close_btn_img {
  border: none;
  width: 14px;
  height: 14px;
  display: inline-block;
  vertical-align: middle;
}

.content_title_title {
  margin-left: 18px;
  margin-top: 8px;
  color: #fff;
  float: left;
  font-size: 18px;
}

.content_content {
  height: 325px;
  position: relative;
  overflow: auto;
}

.ship_info {
  background: #fff;
  height: 325px;
}

.ship_info_main {
  overflow: hidden;
  overflow-y: auto;
}

.ship_info_message {
  padding: 0 20px;
}

.ship_info_tabs {
  padding-top: 14px;
}

.ship_info_message table {
  width: 410px;
  table-layout: fixed;
  border-collapse: collapse;
  border-spacing: 0;
  border: 0;
  display: table;
  box-sizing: border-box;
  text-indent: initial;
}

tbody {
  display: table-row-group;
  vertical-align: middle;
  border-color: inherit;
}

.ship_info th {
  font-weight: 400;
  text-align: right;
  color: #333333;
  font-size: 15px;
  line-height: 24px;
  width: 80px;
  padding: 8px;
}

.ship_info_message td {
  white-space: nowrap;
  font-size: 15px;
  padding: 8px;
}

.location_control {
  background-color: rgba(255, 255, 255, 0.9);
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.25);
  margin: 0;
  color: #333;
  font: 11px/1.5 "helvetica, Neue", Arial, Helvetica, sans-serif;
  margin-bottom: 10px;
  margin-right: 10px;
  margin-left: 10px;
  padding: 5px !important;
  border-radius: 3px;
  width: 110px;
  position: absolute;
  bottom: 0;
  z-index: 800;
  pointer-events: auto;
}

.location_control .lon,
.location_control .lat {
  width: 100%;
  font-size: 12px;
  font-family: "microsoft yahei";
  text-align: left;
  user-select: none;
}

.zoom_shipNum {
  position: absolute;
  bottom: 68px;
  left: 10px;
  text-align: center;
  border-radius: 3px;
  padding: 1px 3px;
  font-size: 12px;
  background-color: rgba(255, 255, 255, 0.63);
  color: #333;
  box-shadow: 2px 2px 6px rgba(0, 0, 0, 0.27);
  width: 115px;
  z-index: 800;
}

.advice_box {
  position: absolute;
  top: 245px;
  right: 12px;
  z-index: 800;
}

.advice_box:hover {
  cursor: pointer;
}

.advice_btn {
  display: block;
  width: 30px;
  height: 30px;
  background: #fff;
  border-radius: 3px;
  text-align: center;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.25);
}

.leaflet_msg {
  width: 400px;
  height: 300px;
  background-color: #fff;
  position: relative;
  overflow: auto;
}

.leave_message {
  color: #999
}

.leave_message li {
  margin-top: 20px;
  line-height: 30px;
  list-style: none;
}

.leave_message span {
  display: inline-block;
  width: 85px;
  text-align: right;
  font-size: 14px;
  margin-right: 10px;
}

.leave_message input {
  width: 210px;
  height: 20px;
}

.leave_message textarea {
  width: 275px;
  height: 150px;
}

.leave_message input,
.leave_message textarea {
  background: #f6f6f6;
  font-size: 12px;
  padding: 4px 10px;
  border: 1px solid #ddd;
  border-radius: 2px;
}

.fullscreen_box {
  position: absolute;
  top: 165px;
  right: 12px;
  z-index: 800;
}

.fullscreen_box:hover {
  cursor: pointer;
}

.fullscreen_btn {
  display: block;
  width: 30px;
  height: 30px;
  background-color: #fff;
  border-radius: 3px;
  text-align: center;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.25);
}

.location_box {
  position: absolute;
  top: 205px;
  right: 12px;
  z-index: 800;
}

.location_box:hover {
  cursor: pointer;
}

.location_btn {
  display: block;
  width: 30px;
  height: 30px;
  background-color: #fff;
  border-radius: 3px;
  text-align: center;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.25);
}

.loc-content {
  // height: 120px;
  position: relative;
  overflow: auto;
}

.loc-content p {
  text-align: center;
  font-size: 18px;
  color: #666;
}

.loc-content input {
  width: 190px;
  height: 28px;
  line-height: 28px;
  border: 1px solid #ddd;
  background: #f6f6f6;
  border-radius: 2px;
  margin: 20px 5px 0;
  font-size: 18px;
  text-align: center;
}

.math_box {
  position: absolute;
  top: 125px;
  right: 12px;
  z-index: 800;
}

.math_box:hover {
  cursor: pointer;
}

.math_btn {
  display: block;
  width: 30px;
  height: 30px;
  background-color: #fff;
  border-radius: 3px;
  text-align: center;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.25);
}

.my-div-icon {
  font-size: 15px;
  padding: 20px;
}

.mapSelect_box {
  background-color: rgba(255, 255, 255, 0.63);
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.25);
  position: absolute;
  margin-left: 10px;
  bottom: 100px;
  width: 120px;
  text-align: center;
  font: 11px/1.5 "helvetica, Neue", Arial, Helvetica, sans-serif;
  color: #333;
  border-radius: 3px;
  z-index: 800;
}

.mapSelect_box .img_box {
  width: 120px;
  height: 80px;
}

.mapSelect_box .img_box:hover {
  border: 1px solid red;
  cursor: pointer;
}

.mapSelect_box img {
  width: 120px;
  height: 80px;
}
</style>
<style>
.el-collapse-item__wrap .el-collapse-item__content {
  padding-bottom: 0px;
}
</style>

