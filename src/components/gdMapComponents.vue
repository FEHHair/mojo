<template>
  <div>
    <div id="container" :style="{ width: widthMap, height: heightMap }"></div>
    <div id="panel" :style="{ maxHeight: panelHeight }"></div>
  </div>
</template>
<script>
//引入缺德地图
import AMapLoader from "@amap/amap-jsapi-loader";

window._AMapSecurityConfig = {
  securityJsCode: "xxxxxx", //你的安全密钥
};
export default {
  props: {
    widthMap: {
      type: String,
      default: "1200px",
    },
    heightMap: {
      type: String,
      default: "800px",
    },
    latitude: {
      type: String,
      default: "29.719251",
    },
    longitude: {
      type: String,
      default: "106.654550",
    },
    shopSocpe: {
      //电子围栏
      type: Array,
      default: () => {
        return [];
      },
    },
  },
  data() {
    return {
      apiGdMap: null,
      m1: null,
      m2: null,
    };
  },
  computed: {
    panelHeight() {
      let height = this.heightMap.replace(/px/g, "");
      return height - 20 + "px";
    },
  },
  mounted() {
    //DOM初始化完成进行地图初始化
    this.initMapLoad();
  },
  beforeDestroy() {
    // 离开页面前需要执行的操作
    this.apiGdMap && this.apiGdMap.destroy();
    console.log("地图已销毁");
  },
  methods: {
    initMapLoad() {
      let load = AMapLoader.load({
        key: "xxxxxxxxxxxxx", // 申请好的Web端开发者Key，首次调用 load 时必填
        version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
        plugins: [
          "AMap.PlaceSearch",
          "AMap.Geocoder",
          "AMap.Polygon",
          "AMap.LngLat",
        ],
      })
        .then(() => {
          this.initMap();
        })
        .catch((e) => {
          console.log(e);
        });
      console.log(load);
    },
    initMap() {
      this.apiGdMap = new AMap.Map("container", {
        resizeEnable: true,
        viewMode: "2D", //默认使用 2D 模式
        center: [this.longitude, this.latitude], //地图中心点
        zoom: 13, //地图级别
      });
      // 创建一个默认的点标记
      this.getPoints(this.longitude, this.latitude);
      if (this.shopSocpe && this.shopSocpe.length) {
        //画电子围栏
        this.setMarkerMore(this.shopSocpe);
      }
    },
    //创建点标记
    getPoints(longitude, latitude, type = 1) {
      this.clearMarkerM1();
      // 创建点标记
      this.m1 = new AMap.Marker({
        position: [longitude, latitude],
      });
      this.apiGdMap.add(this.m1);
      if (type != 1) {
        //绘制地址范围才调用
        this.getPointHui(longitude, latitude);
      }
    },
    //设置中心点坐标
    setCenter(longitude, latitude) {
      this.apiGdMap.setCenter([longitude, latitude]);
    },
    //根据坐标逆地址解析
    getGeocoder(longitude, latitude) {
      let that = this;
      let geocoder = new AMap.Geocoder({
        city: "023", //城市设为北京，默认：“全国”
        radius: 1000, //范围，默认：500
      });
      geocoder.getAddress([longitude, latitude], function (status, result) {
        if (status === "complete" && result.regeocode) {
          let address = result.regeocode.formattedAddress;
          console.log("address", address);
          return address;
        } else {
          that.$message.error("根据经纬度查询地址失败");
        }
      });
    },
    setMarkerMore(data) {
      this.clearMarkerM2();
      let polygonPath = data.map((item) => {
        return new AMap.LngLat(item.y, item.x);
      });
      // console.log('polygonPath',polygonPath)
      // 创建面覆盖物
      this.m2 = new AMap.Polygon({
        path: polygonPath,
      });

      this.apiGdMap.add(this.m2);
    },
    //绘制地址范围
    getPointHui(longitude, latitude) {
      //根据坐标逆地址解析
      let that = this;
      let geocoder = new AMap.Geocoder({
        city: "023", //城市设为北京，默认：“全国”
        radius: 1000, //范围，默认：500
      });
      geocoder.getAddress([longitude, latitude], function (status, result) {
        if (status === "complete" && result.regeocode) {
          let address = result.regeocode.formattedAddress;
          //设置中心点坐标
          that.setCenter(longitude, latitude);
          let data = {
            latitude,
            longitude,
            address,
          };
          console.log("绘制地址范围", data);
          that.clickMap(data);
        } else {
          that.$message.error("根据经纬度查询地址失败");
        }
      });
    },
    // 清除 m1
    clearMarkerM1() {
      if (this.m1) {
        this.m1.setMap(null);
        this.m1 = null;
      }
    },
    // 清除 marker
    clearMarkerM2() {
      if (this.m2) {
        this.m2.setMap(null);
        this.m2 = null;
      }
    },
    getSearch(address) {
      console.log("搜索地址", address);
      let that = this;
      let map = this.apiGdMap;
      //构造地点查询类
      var placeSearch = new AMap.PlaceSearch({
        city: "023",
        pageSize: 20, // 单页显示结果条数
        pageIndex: 1, // 页码
        zoom: 15, //地图显示的缩放级别
        map, // 展现结果的地图实例
        panel: "panel", // 结果列表将在此容器中进行展示。
        autoFitView: true, // 是否自动调整地图视野使绘制的 Marker点都处于视口的可见范围
      });
      //关键字查询方法
      placeSearch.search(address);
      // 添加listElementClick事件监听器
      placeSearch.on("listElementClick", function (event) {
        console.log("点击获取信息", event.data);
        that.clearMarkerM1();
        // 获取被点击的列表项的相关信息
        let data = {
          latitude: event.data.location.lat,
          longitude: event.data.location.lng,
          address: event.data.address,
        };
        that.clickMap(data);
      });
    },
    clickMap(data) {
      this.$emit("clickMap", {
        latitude: data.latitude,
        longitude: data.longitude,
        address: data.address,
      });
    },
  },
};
</script>
<style lang="scss" scoped>
#panel {
  position: absolute;
  background-color: white;
  overflow-y: auto;
  top: 0px;
  left: 0px;
  width: 280px;
}
</style>
