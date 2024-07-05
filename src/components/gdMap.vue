<template>
  <div id="container" :style="{ width: widthMap, height: heightMap }"></div>
  <div id="panel" :style="{ maxHeight: panelHeight }"></div>
</template>

<script setup lang="ts">
//npm i @amap/amap-jsapi-loader --save-dev 引入缺德地图
import {defineProps, onMounted, beforeUnmount, ref} from "vue";
import AMapLoader from "@amap/amap-jsapi-loader";

window._AMapSecurityConfig = {
  securityJsCode: "xxxxxx", //你的安全密钥
};

const props = defineProps({
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
})


const apiGdMap = ref(null);
const m1 = ref(null);
const m2 = ref(null);

const initMapLoad = async () => {
  try {
    await AMapLoader.load({
      key: "xxxxxxxxxxxxx", // 申请好的Web端开发者Key，首次调用 load 时必填
      version: "2.0", // 指定要加载的 JSAPI 的版本，缺省时默认为 1.4.15
      plugins: [
        "AMap.PlaceSearch",
        "AMap.Geocoder",
        "AMap.Polygon",
        "AMap.LngLat",
      ],
    });
    initMap();
  } catch (error) {
    console.log(error);
  }
};

const initMap = () => {
  apiGdMap.value = new AMap.Map("container", {
    resizeEnable: true,
    viewMode: "2D", //默认使用 2D 模式
    center: [props.longitude, props.latitude], //地图中心点
    zoom: 13, //地图级别
  });
  // 创建一个默认的点标记
  getPoints(props.longitude, props.latitude);
  if (props.shopSocpe && props.shopSocpe.length) {
    //画电子围栏
    setMarkerMore(props.shopSocpe);
  }
};

const getPoints = (longitude, latitude, type = 1) => {
  clearMarkerM1();
  // 创建点标记
  m1.value = new AMap.Marker({
    position: [longitude, latitude],
  });
  apiGdMap.value.add(m1.value);
  if (type != 1) {
    //绘制地址范围才调用
    getPointHui(longitude, latitude);
  }
};

const setCenter = (longitude, latitude) => {
  apiGdMap.value.setCenter([longitude, latitude]);
};

const getGeocoder = (longitude, latitude) => {
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
};

const setMarkerMore = (data) => {
  clearMarkerM2();
  let polygonPath = data.map((item) => {
    return new AMap.LngLat(item.y, item.x);
  });
  // 创建面覆盖物
  m2.value = new AMap.Polygon({
    path: polygonPath,
  });

  apiGdMap.value.add(m2.value);
};

const getPointHui = (longitude, latitude) => {
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
      setCenter(longitude, latitude);
      let data = {
        latitude,
        longitude,
        address,
      };
      console.log("绘制地址范围", data);
      clickMap(data);
    } else {
      that.$message.error("根据经纬度查询地址失败");
    }
  });
};

const clearMarkerM1 = () => {
  if (m1.value) {
    m1.value.setMap(null);
    m1.value = null;
  }
};

const clearMarkerM2 = () => {
  if (m2.value) {
    m2.value.setMap(null);
    m2.value = null;
  }
};

const getSearch = (address) => {
  console.log("搜索地址", address);
  let that = this;
  let map = apiGdMap.value;
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
    clearMarkerM1();
    // 获取被点击的列表项的相关信息
    let data = {
      latitude: event.data.location.lat,
      longitude: event.data.location.lng,
      address: event.data.address,
    };
    clickMap(data);
  });
};

const clickMap = (data) => {
  emit("clickMap", {
    latitude: data.latitude,
    longitude: data.longitude,
    address: data.address,
  });
};

onMounted(() => {
  initMapLoad();
});

beforeUnmount(() => {
  apiGdMap.value && apiGdMap.value.destroy();
  console.log("地图已销毁");
});

return {
  apiGdMap,
  m1,
  m2,
  initMapLoad,
  initMap,
  getPoints,
  setCenter,
  getGeocoder,
  setMarkerMore,
  getPointHui,
  clearMarkerM1,
  clearMarkerM2,
  getSearch,
  clickMap,
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
