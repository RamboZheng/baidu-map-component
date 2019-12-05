<template>
  <baidu-map
    class="map"
    id="container"
    :zoom="nearby.zoom"
    :center="nearby.center"
    @ready="handler"
    :scroll-wheel-zoom="true"
    :keyboard="false"
  >
    <!-- 画圆 -->
    <bm-circle
      :center="nearby.center"
      :radius="nearby.radius"
      stroke-color="blue"
      strokeColor="#6595F4"
      :stroke-opacity="0.5"
      :stroke-weight="1"
      fillColor="#6595F4"
      :fillOpacity="0.2"
    ></bm-circle>
    <!-- 中心点 -->
    <bm-marker
      :position="nearby.center"
      :dragging="false"
      :icon="{url: '中心icon地址', size: {width: 28, height: 36}}"
    ></bm-marker>
    <!-- 周边检索 -->
    <bm-marker
      v-for="(item,index) in pointList"
      :key="index"
      :position="item.location"
      :dragging="false"
      :title="item.name"
      @click="lookDetail(item)"
    >
      <!-- 点上的序号 -->
      <bm-label
        :content="index<10?'0'+index:''+index"
        :labelStyle="{color: 'white', fontSize : '1px',backgroundColor:'red',border:'none',borderRadius:'50%'}"
        :offset="{width: 1,height: 2 }"
      />
    </bm-marker>
    <!-- 信息弹窗 -->
    <bm-info-window
      class="map-info"
      :position="{lng: nearby.center.lng, lat: nearby.center.lat}"
      :show="infoWindow.show"
      :width="30"
      :autoPan="true"
      :offset="{width: 1,height: -20}"
      @close="infoWindowClose"
      @open="infoWindowOpen"
    >
      <!-- 弹窗图标/题目/距离/地址 -->
      <img src="地图点提示icon" />
      <p class="map-title" v-text="infoWindow.name"></p>
      <p class="map-distance" v-text="infoWindow.distance+'米'"></p>
      <p class="map-address" v-text="infoWindow.address"></p>
    </bm-info-window>
    <!-- 信息列表 -->
    <el-tabs
      class="info-list"
      type="border-card"
      v-model="pointType"
      :stretch="true"
      @tab-click="changeList"
    >
      <el-tab-pane label="交通" name="traffic" style="padding:0">
        <!-- 切换按钮 -->
        <el-radio-group v-model="trafficType" style="margin:22px 0" @change="changeTraffic">
          <el-radio-button label="公交" name="bus">公交</el-radio-button>
          <el-radio-button label="地铁" name="metro">地铁</el-radio-button>
        </el-radio-group>
        <div ref="traffic" class="point-box" v-if="pointType=='traffic'" style="max-height: 400px;">
          <div
            v-for="item in pointList"
            :key="item.uid"
            @click="lookDetail(item)"
            :class="{active: activeName == item.name}"
            class="point-item"
          >
            <div class="point-top">
              <img :src="trafficType=='公交'?iconUrl.bus:iconUrl.metro" />
              {{item.name}}
              <span class="point-distance">{{item.distance}}米</span>
            </div>

            <div class="point-bottom">{{item.address}}</div>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="教育" name="aducate">
        <div ref="aducate" class="point-box" v-if="pointType=='aducate'" style="max-height: 500px;">
          <div
            v-for="item in pointList"
            :key="item.uid"
            @click="lookDetail(item)"
            :class="{active: activeName == item.name}"
            class="point-item"
          >
            <div class="point-top">
              <img :src="iconUrl.aducate" />
              {{item.name}}
              <span class="point-distance">{{item.distance}}米</span>
            </div>
            <div class="point-bottom">{{item.address}}</div>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="医疗" name="medical">
        <div ref="medical" class="point-box" v-if="pointType=='medical'" style="max-height: 500px;">
          <div
            v-for="item in pointList"
            :key="item.uid"
            @click="lookDetail(item)"
            :class="{active: activeName == item.name}"
            class="point-item"
          >
            <div class="point-top">
              <img :src="iconUrl.medical" />
              {{item.name}}
              <span class="point-distance">{{item.distance}}米</span>
            </div>
            <div class="point-bottom">{{item.address}}</div>
          </div>
        </div>
      </el-tab-pane>
      <el-tab-pane label="生活" name="living">
        <div ref="living" class="point-box" v-if="pointType=='living'" style="max-height: 500px;">
          <div
            v-for="item in pointList"
            :key="item.uid"
            @click="lookDetail(item)"
            :class="{active: activeName == item.name}"
            class="point-item"
          >
            <div class="point-top">
              <img :src="iconUrl.living" />
              {{item.name}}
              <span class="point-distance">{{item.distance}}米</span>
            </div>
            <div class="point-bottom">{{item.address}}</div>
          </div>
        </div>
      </el-tab-pane>
    </el-tabs>
  </baidu-map>
</template>

<script>
import API from "@/axios/api/commonAPI";
export default {
  props: {
    gardenLatitude: {
      type: Number
    },
    gardenLongitude: {
      type: Number
    }
  },
  data() {
    return {
      // 图标地址
      iconUrl: {
        bus: "列表icon地址",
        metro: "列表icon地址",
        aducate: "列表icon地址",
        medical: "列表icon地址",
        living: "列表icon地址"
      },
      // 当前选中item
      activeName: "",
      // 图标信息
      infoWindow: {
        location: { lng: 113.934857, lat: 22.517526 }
      },
      showInfo: false,
      // 交通类型
      trafficType: "公交",
      // 检索类型
      pointType: "traffic",
      // 周边数组
      pointList: [],
      // 附近信息
      nearby: {
        center: {
          lng: 113.934857,
          lat: 22.517526
        },
        zoom: 15,
        radius: 1000
      }
    };
  },
  methods: {
    // 地图初始化以后的回调
    handler() {
      this.nearby.center.lat = this.gardenLatitude;
      this.nearby.center.lng = this.gardenLongitude;
      this.getMapAround(this.trafficType);
    },

    //打开图标信息弹窗
    infoWindowOpen() {},
    // 关闭图标信息弹窗
    infoWindowClose() {},
    //点击详情
    lookDetail(data) {
      this.activeName = data.name;
      this.infoWindow = data;
      this.infoWindow.show = true;
      this.$nextTick(() => {
        let obj = document.querySelector(".active");
        let scrollTop = obj.offsetTop;
        switch (this.pointType) {
          case "traffic":
            this.$refs.traffic.scrollTop = scrollTop - 180;
            break;
          case "aducate":
            this.$refs.aducate.scrollTop = scrollTop - 380;
            break;
          case "medical":
            this.$refs.medical.scrollTop = scrollTop - 180;
            break;
          case "living":
            this.$refs.living.scrollTop = scrollTop - 180;
            break;
        }
      });
    },
    // 更改地图类型
    changeList(val) {
      this.infoWindow.show = false;
      if (val.name == "traffic") {
        this.getMapAround(this.trafficType);
      } else {
        this.getMapAround(val.label);
      }
    },
    // 更改交通方式
    changeTraffic(val) {
      this.infoWindow.show = false;
      this.getMapAround(val);
    },
    // 获取周边数据
    getMapAround(val) {
      this.pointList = [];
      let ctx = this,
        location = this.nearby.center.lat + "," + this.nearby.center.lng,
        parme = {
          location: location,
          query: val,
          radius: 1000,
          output: "json",
          ak: "百度ad",
          page_size: 50
        };
      // 为了防止打包以后跨越 百度周边列表数据请求使用jsonp
      this.$jsonp("http://api.map.baidu.com/place/v2/search", parme).then(
        result => {
          ctx.pointList = result.results;
          for (let item of ctx.pointList) {
            let tag = JSON.parse(JSON.stringify(item.location));
            let center = JSON.parse(JSON.stringify(ctx.nearby.center));
            item.distance = ctx.distance(center, tag);
          }
        }
      );
    },
    // 计算两点距离
    distance(center, tag) {
      let map = new BMap.Map("allmap");
      let pointA = new BMap.Point(center.lng, center.lat); // 创建点坐标A
      let pointB = new BMap.Point(tag.lng, tag.lat); // 创建点坐标B
      return map.getDistance(pointA, pointB).toFixed(0);
    }
  }
};
</script>
<style lang='scss' scoped>
// 隐藏滚动条

.map {
  width: 580px;
  height: 560px;
  .point-box {
    overflow-y: scroll;
    overflow-x: hidden;
    .point-item {
      padding-bottom: 20px;
      .point-top {
        padding-right: 48px;
        font-size: 16px;
        color: #475266;
        position: relative;
        .point-distance {
          position: absolute;
          right: 0;
          top: 0;
          font-size: 16px;
          color: #878d99;
        }
        img {
          margin-right: 5px;
          width: 13px;
          height: 14px;
          vertical-align: middle;
        }
      }
      .point-bottom {
        margin-top: 10px;
        font-size: 16px;
        color: #cbcbcb;
      }
    }
    .active {
      .point-top {
        color: #6595f4;
      }
    }
  }
  .map-info {
    img {
      width: 13px;
      height: 14px;
      position: absolute;
      top: 20px;
      left: 0;
    }
    .map-title {
      padding-left: 20px;
      display: inline-block;
      font-size: 16px;
      color: #475266;
      width: 155px;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
    }
    .map-distance {
      font-size: 16px;
      color: #878d99;
      position: absolute;
      top: 0;
      right: 0;
    }
    .map-address {
      font-size: 16px;
      color: #cbcbcb;
    }
  }
  // 列表样式
  .info-list {
    position: absolute;
    top: 137px;
    right: 0;
    width: 360px;
    height: 560px;
  }
  .point-box::-webkit-scrollbar {
    display: none;
  }
}

// 清除列表标签默认样式
.el-tabs--border-card {
  border: none;
  -webkit-box-shadow: none;
  box-shadow: none;
}
</style>




