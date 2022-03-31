### 页面代码

```vue
<template>
  <div class="page-content">
    <div id="container" ref="Container" tabindex="0"/>
    <!--组织架构-->
    <el-card class="tree-card" shadow="always" :body-style="{padding: '10px'}">
      <div class="tree-card-nav">
        <el-radio-group v-model="searchObj.type" size="mini">
          <el-radio-button :label="0">在途监管</el-radio-button>
          <el-radio-button :label="1">全部派单</el-radio-button>
        </el-radio-group>
        <i :class="isTreeList ? 'el-icon-arrow-up' : 'el-icon-arrow-down'" style="font-size: 20px;" @click="cutoverHandle"/>
      </div>
      <div v-show="isTreeList" class="tree-card-list">
        <el-input v-model="filterText" size="mini" placeholder="输入关键字进行过滤"/>
        <el-tree
          ref="fenceTree"
          :data="monitorList"
          :props="defaultProps"
          default-expand-all
          :filter-node-method="filterNode"
        />
      </div>
    </el-card>
    <!--地图查询-->
    <el-card class="search-card" shadow="always" :body-style="{padding: '10px'}">
      <el-badge :is-dot="alarmList.length > 0" class="item">
        <el-button icon="el-icon-message-solid" size="mini" @click="alarmHandle">报警</el-button>
      </el-badge>
      <el-button class="item" icon="el-icon-position" :disabled="isRanging" size="mini" @click="rangingHandle">测距
      </el-button>
      <el-input v-model="cityNameCode" class="item" placeholder="搜索城市" prefix-icon="el-icon-search" size="mini" style="width: 200px" @change="cityHandle">
        <el-button slot="append" icon="el-icon-search" @click="cityHandle"/>
      </el-input>
    </el-card>
    <!--事件类型-->
    <el-card class="type-card" shadow="always" :body-style="{padding: '10px'}">
      <el-badge :value="12">
        <el-button size="mini" type="success">正常</el-button>
      </el-badge>
      <el-badge :value="3">
        <el-button size="mini" type="danger">紧急</el-button>
      </el-badge>
      <el-badge :value="1">
        <el-button size="mini" type="warning">告警</el-button>
      </el-badge>
      <el-badge :value="2">
        <el-button size="mini" type="info">预警</el-button>
      </el-badge>
    </el-card>
    <!--监管信息-->
    <el-card v-show="isDetail" class="info-card" shadow="always" :body-style="{padding: '10px'}">
      <div class="info-card-nav">
        <div>监管信息</div>
        <i :class="isInfoList ? 'el-icon-arrow-up' : 'el-icon-arrow-down'" style="font-size: 20px;" @click="cutoverInfoHandle"/>
      </div>
      <div v-show="isInfoList" class="info-card-list">
        <div>送货单号: xxxxxxx</div>
        <div>路线名称: 创新大厦--中山公园</div>
        <el-divider/>
        <div>锁状态: 开锁/关锁</div>
        <div>锁绳状态: 插入/拔出</div>
        <div>GPS信号: 8</div>
        <div>GSM信号: 50</div>
        <div>电量: 80%</div>
        <div>定位类型：小区码定位、不定位、GPS定位</div>
        <div>当前位置：xxxxxxxxx</div>
        <div>经度：xxxxxxxxx</div>
        <div>纬度：xxxxxxxxx</div>
        <div>更新时间：2022-3-14 14:00:00</div>
        <div>小区码：xxxxxx</div>
        <div>运行状态：停车【10时10分】</div>
        <div>速度：32km/h</div>
        <el-divider/>
        <div>主驾司机：张三 15912341234</div>
        <div>副驾司机：李四 15912341234</div>
        <div>车牌号：xxxxx</div>
        <div>车厢号：xxxxx</div>
        <div>设备ID: xxxxxxx</div>
        <div>实际发出时间：2022-03-28 13:00:00</div>
        <div>实际达到时间：2022-03-29 13:00:00</div>
      </div>
    </el-card>
    <!--报警列表-->
    <el-card v-show="isAlarm" class="alarm-card" shadow="always" :body-style="{padding: '10px'}">
      <el-table v-loading="alarmLoading" :data="alarmList" border stripe size="small" :height="240">
        <el-table-column label="序号" type="index" width="50" align="center" />
        <el-table-column prop="deviceName" label="报警时间" align="center" />
        <el-table-column prop="deviceName" label="报警名称" align="center" />
        <el-table-column prop="deviceName" label="报警等级" align="center" />
        <el-table-column prop="deviceName" label="设备ID" align="center" />
        <el-table-column prop="deviceName" label="车厢号" align="center" />
        <el-table-column prop="deviceName" label="货运单号" align="center" />
        <el-table-column prop="deviceName" label="操作" align="center" />
      </el-table>
      <el-pagination
        :current-page="pageInfo.pageNo"
        :page-size="pageInfo.pageSize"
        layout="total, prev, pager, next, jumper"
        :total="pageInfo.total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'TimeMonitor',
  data() {
    return {
      MAP: null,
      Ranging: null,
      isRanging: false,
      drivingObj: null,
      isDetail: false,
      points: [],
      /* 货运单查询 */
      cityNameCode: '',
      filterText: '',
      defaultProps: {
        children: 'children',
        label: 'label'
      },
      isTreeList: true,
      isInfoList: true,
      monitorList: [
        {
          id: 1,
          label: '集团公司名称',
          children: [{
            id: 4,
            label: '子公司一',
            children: [{
              id: 9,
              label: '产品研发中心',
              children: [{
                id: 9,
                label: '送货单号001'
              }, {
                id: 10,
                label: '送货单号002'
              }]
            }]
          }, {
            id: 4,
            label: '子公司二',
            children: [{
              id: 9,
              label: '销售部',
              children: [{
                id: 9,
                label: '送货单号001'
              }, {
                id: 10,
                label: '送货单号002'
              }]
            }]
          }]
        }
      ],
      searchObj: {
        type: 0
      },
      // 报警列表
      isAlarm: false,
      alarmLoading: false,
      pageInfo: {
        pageNo: 1,
        pageSize: 5,
        total: 0
      },
      alarmList: []
    }
  },
  watch: {
    filterText(val) {
      this.$refs.tree.filter(val)
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initMap()
    })
  },
  destroyed() {
    this.MAP && this.MAP.destroy()
  },
  methods: {
    initMap() {
      setTimeout(() => {
        // 使用 $refs 获取节点
        this.MAP = new this.$AMap.Map(this.$refs.Container, {
          resizeEnable: true,
          zoom: 16
        })
        // 在图面添加工具条控件，工具条控件集成了缩放、平移、定位等功能按钮在内的组合控件
        this.MAP.addControl(new this.$AMap.ToolBar({ position: { bottom: '50px', left: '10px' }}))
        // 在图面添加比例尺控件，展示地图在当前层级和纬度下的比例尺
        this.MAP.addControl(new this.$AMap.Scale())
        // 测距工具
        this.Ranging = new this.$AMap.RangingTool(this.MAP)
        this.Ranging.on('end', () => {
          this.isRanging = false
          this.Ranging.turnOff()
        })
        // 构造路线导航类
        this.drivingObj = new this.$AMap.Driving({
          policy: this.$AMap.DrivingPolicy.LEAST_TIME, // 最快捷模式
          ferry: 1 // 是否可以使用轮渡
        })
        this.MAP.on('click', () => {
          this.isAlarm = false
          this.isDetail = false
        })
        this.MAP.on('complete', () => {
          // 在查询成功后，调用点聚合的初始化功能
          this.initMarkers()
        })
      }, 600)
    },
    /* 清除地图上所有覆盖物 */
    removeAllOverlay() {
      this.MAP.clearMap()
    },
    /* 测距工具 */
    rangingHandle() {
      this.isRanging = true
      this.Ranging.turnOn()
    },
    // 根据cityName、adCode、cityCode设置地图位置
    gotoCity() {
      if (!this.cityNameCode) {
        this.cityNameCode = '深圳市'
      }
      this.MAP.setCity(this.cityNameCode)
    },
    /* 货单过滤 */
    filterNode(value, data) {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    },
    // 切换tree的显示隐藏
    cutoverHandle() {
      this.isTreeList = !this.isTreeList
    },
    // 查询城市
    cityHandle() {
      this.gotoCity()
    },
    // 监管信息详情
    cutoverInfoHandle() {
      this.isInfoList = !this.isInfoList
    },
    // 添加点聚合
    initMarkers() {
      // 初始化聚合列表和点的列表
      const gridSize = 60
      let cluster
      const that = this
      const circlesList = [
        {
          center: [114.020064, 22.520469],
          radius: 500
        }, {
          center: [114.081283, 22.524671],
          radius: 800
        }
      ]
      const points = [
        {
          lnglat: [114.035084, 22.527446],
          id: 1,
          level: 0
        }, {
          lnglat: [114.021609, 22.527961],
          id: 2,
          level: 1
        }, {
          lnglat: [114.026029, 22.529071],
          id: 3,
          level: 2
        }, {
          lnglat: [114.063655, 22.528799],
          id: 4,
          level: 3
        }, {
          lnglat: [114.084239, 22.532247],
          id: 5,
          level: 0
        }
      ]
      const count = points.length
      const _renderClusterMarker = function(context) {
        const factor = Math.pow(context.count / count, 1 / 18)
        const div = document.createElement('div')
        const Hue = 180 - factor * 180
        const bgColor = 'hsla(' + Hue + ',100%,40%,0.7)'
        const fontColor = 'hsla(' + Hue + ',100%,90%,1)'
        const borderColor = 'hsla(' + Hue + ',100%,40%,1)'
        const shadowColor = 'hsla(' + Hue + ',100%,90%,1)'
        div.style.backgroundColor = bgColor
        const size = Math.round(30 + Math.pow(context.count / count, 1 / 5) * 20)
        div.style.width = div.style.height = size + 'px'
        div.style.border = 'solid 1px ' + borderColor
        div.style.borderRadius = size / 2 + 'px'
        div.style.boxShadow = '0 0 5px ' + shadowColor
        div.innerHTML = context.count
        div.style.lineHeight = size + 'px'
        div.style.color = fontColor
        div.style.fontSize = '14px'
        div.style.textAlign = 'center'
        context.marker.setOffset(new that.$AMap.Pixel(-size / 2, -size / 2))
        context.marker.setContent(div)
      }
      const _renderMarker = function(context) {
        const level = context.data[0].level
        let iconColor = ''
        switch (level) {
          case 0:
            iconColor = '#F56C6C'
            break
          case 1:
            iconColor = '#E6A23C'
            break
          case 2:
            iconColor = '#f4ea2a'
            break
          case 3:
            iconColor = '#67C23A'
            break
        }
        const content = `<i class="iconfont icon-huoche" style="color: ${iconColor};"></i>`
        const offset = new that.$AMap.Pixel(-15, -15)
        context.marker.setContent(content)
        context.marker.setOffset(offset)
        context.marker.setExtData({
          id: context.data[0].id
        })
        // 此处为添加单个标记点击事件
        context.marker.on('click', ev => {
          // 当前标记居中
          // that.MAP.setZoomAndCenter(16, ev.target.getPosition())
          // 获取标记携带的数据
          const extData = ev.target.getExtData()
          console.log(extData)
          that.isDetail = true
        })
      }
      // 调用函数添加点聚合功能
      addCluster()

      function addCluster() {
        if (cluster) {
          cluster.setMap(null)
        }
        cluster = new that.$AMap.MarkerClusterer(that.MAP, points, {
          gridSize: gridSize,
          renderClusterMarker: _renderClusterMarker, // 自定义聚合点样式
          renderMarker: _renderMarker // 自定义非聚合点样式
        })
      }

      cluster.on('click', item => {
        // 此处是通过包含点的数量判断是否是聚合点，不是聚合点就执行上方单个点的点击方式
        if (item.clusterData.length <= 1) {
          return
        }
        // 这里是计算所有聚合点的中心点
        let alllng = 0
        let alllat = 0
        for (const mo of item.clusterData) {
          alllng += mo.lnglat.lng
          alllat += mo.lnglat.lat
        }
        const lat = alllat / item.clusterData.length
        const lng = alllng / item.clusterData.length
        // 这里是放大地图，此处写死了每次点击放大的级别，可以根据点的数量和当前大小适应放大，体验更佳
        that.MAP.setZoomAndCenter(that.MAP.getZoom() + 2, [lng, lat])
      })
      // 绘制围栏
      for (const item of circlesList) {
        const circle = new that.$AMap.Circle({
          center: item.center,
          radius: item.radius, // 半径
          borderWeight: 3,
          strokeColor: '#FF33FF',
          strokeWeight: 6,
          strokeOpacity: 0.2,
          fillOpacity: 0.4,
          strokeStyle: 'solid',
          strokeDasharray: [10, 10],
          // 线样式还支持 'dashed'
          fillColor: '#1791fc',
          zIndex: 50
        })
        setCirclePoint(item.center, item.radius, circle)
        this.MAP.add(circle)
      }
      // 根据起终点经纬度规划驾车导航路线
      this.drivingObj.search(new that.$AMap.LngLat(114.020064, 22.520469), new that.$AMap.LngLat(114.081283, 22.524671), function(status, result) {
        // result即是对应的驾车导航信息
        if (status === 'complete') {
          if (result.routes && result.routes.length) {
            // 绘制第一条路线，也可以按需求绘制其它几条路线
            drawRoute(result.routes[0])
          }
        } else {
          this.$message.info('获取驾车数据失败：' + result)
        }
      })

      function drawRoute(route) {
        const path = parseRouteToPath(route)
        const startMarker = new that.$AMap.Marker({
          position: path[0],
          icon: 'https://webapi.amap.com/theme/v1.3/markers/n/start.png',
          map: that.MAP
        })

        const endMarker = new that.$AMap.Marker({
          position: path[path.length - 1],
          icon: 'https://webapi.amap.com/theme/v1.3/markers/n/end.png',
          map: that.MAP
        })
        const routeLine = new that.$AMap.Polyline({
          path: path,
          isOutline: true,
          outlineColor: '#ffeeee',
          borderWeight: 2,
          strokeWeight: 5,
          strokeOpacity: 0.9,
          strokeColor: '#0091ff',
          lineJoin: 'round'
        })
        that.MAP.add(routeLine)
        // 调整视野达到最佳显示区域
        that.MAP.setFitView([startMarker, endMarker, routeLine])
      }
      function parseRouteToPath(route) {
        const path = []
        for (let i = 0, l = route.steps.length; i < l; i++) {
          const step = route.steps[i]
          for (let j = 0, n = step.path.length; j < n; j++) {
            path.push(step.path[j])
          }
        }
        return path
      }
      /**
       *
       * 获取圆上的点（步长 45度）
       *
       **/
      function setCirclePoint(centerpoint, radius, circle) {
        const r = 6371000 // 地球的平均半径
        const numpoints = 360
        const phase = 2 * Math.PI / numpoints
        let points = null

        // 画点
        // 计算坐标点
        const dx = (radius * Math.cos(45 * phase))
        const dy = (radius * Math.sin(45 * phase))
        // 转换成经纬度
        const dlng = dx / (r * Math.cos(centerpoint[1] * Math.PI / 180) * Math.PI / 180)
        const dlat = dy / (r * Math.PI / 180)
        const newlng = centerpoint[0] + dlng
        const newlag = centerpoint[1] + dlat
        points = [newlng, newlag]
        // 实例化点标记
        const contentIcon = `<i class="iconfont icon-close"></i>`
        const marker = new that.$AMap.Marker({
          position: points,
          content: contentIcon,
          offset: new that.$AMap.Pixel(-10, -10)
        })
        marker.on('click', e => {
          console.log(e)
          circle.hide()
          marker.hide()
        })
        marker.setMap(that.MAP)
        return points
      }
    },
    // 报警列表
    alarmHandle() {
      this.isAlarm = !this.isAlarm
    },
    handleSizeChange(val) {
      this.pageInfo.pageSize = val
    },
    handleCurrentChange(val) {
      this.pageInfo.pageNo = val
    }
  }
}
</script>

<style lang="scss" scoped>
@import "//at.alicdn.com/t/font_3215613_lv5jyvtei8b.css";

.page-content {
  width: 100%;
  height: calc(100vh - 50px);
  position: relative;
}

#container {
  width: 100%;
  height: 100%;
}

.search-card {
  position: absolute;
  top: 10px;
  left: 270px;

  .item {
    margin: 0 10px;
  }
}

.type-card {
  position: absolute;
  top: 10px;
  right: 270px;

  .el-badge {
    margin: 0 10px;
  }
}

.tree-card {
  width: 250px;
  position: absolute;
  left: 10px;
  top: 10px;

  &-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;

    .el-icon-arrow-down {
      cursor: pointer;
    }
  }

  &-list {
    margin-top: 10px;
  }
}

.info-card {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 250px;
  font-size: 12px;
  line-height: 1.8;

  &-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 28px;

    .el-icon-arrow-down {
      cursor: pointer;
    }
  }
}

::v-deep .icon-huoche {
  font-size: 30px;
  background-color: #999;
  border-radius: 50%;

  /* &::after {
    content: '';
    height: 0;
    width: 0;
    border: 10px transparent solid;
    display: block;
    position: absolute;
    bottom: -17.5px;
    left: 15px;
    transform: translateX(-50%);
    border-right: 10px solid transparent;
    border-top: 15px solid #999;
    border-left: 10px solid transparent;
  } */
}
::v-deep .icon-close {
  font-size: 18px;
  color: #606266;
  border-radius: 50%;
}
.alarm-card {
  position: absolute;
  top: 68px;
  left: 270px;
}
.el-pagination {
  float: right;
}
</style>

```

### 参考

> https://www.cnblogs.com/seemoon/p/12916516.html
>
> https://blog.csdn.net/qq_39157025/article/details/120287561
>
> http://nicethemes.cn/news/txtlist_i233613v.html
>
> https://blog.csdn.net/qq_39157025/article/details/120287561
>
> https://segmentfault.com/a/1190000024509848
