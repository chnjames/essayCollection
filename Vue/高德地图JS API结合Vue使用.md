1. **开发准备**

   1. 注册开发者账号，成为高德开放平台开发者
   2. 登陆之后，在进入 `应用管理` 页面 `创建新应用`
   3. 为应用 `添加Key`，`服务平台`一项选择 `Web端(JSAPI)`
   4. 添加成功后，获取到Key值

2. **地图组件开发和使用**

   获取高德地图API，可通过JavaScript脚本的方式加载和官网提供的JSAPI Loader为了更好的契合Vue使用推荐第二种方式。

   1. 按NPM方式安装使用Loader：

      ```sh
      npm i @amap/amap-jsapi-loader --save
      ```

      

   2. 在项目中新建 MapContainer.vue 文件，用作地图组件。

   3. 在 MapContainer.vue 地图组件中创建 div 标签作为地图容器 ，并设置地图容器的 id 属性为 container；

      <template>
        <div id="container"></div>
      </template>

   4. 设置地图容器样式

      ```css
      <style scoped>
          #container{
              padding:0px;
              margin: 0px;
              width: 100%;
              height: 800px;
          }
      </style>
      ```

      

   5. 在utils下新建amap.js文件

      ```javascript
      import AMapLoader from '@amap/amap-jsapi-loader'
      
      const install = Vue => {
        window._AMapSecurityConfig = {
          securityJsCode: 'xxxxx'
        }
        AMapLoader.load({
          key: 'xxxxxx',
          version: '2.0',
          plugins: [
            'AMap.InfoWindow', // 信息窗体
            'AMap.ContextMenu', // 覆盖物地图右键菜单
            'AMap.MouseTool', // 绘制覆盖物
            'AMap.RangingTool', // 测距工具
            'AMap.CircleEditor', // 圆形编辑器
            'AMap.PolyEditor', // 多边形编辑工具
            'AMap.OverView', // 鹰眼
            'AMap.MapType', // 地图类型切换
            'AMap.MoveAnimation', // 轨迹回放
            'AMap.ToolBar', // 地图控件
            'AMap.Driving', // 路径规划
            'AMap.Scale', // 比例尺插件
            'AMap.Geolocation', // 高精度定位服务
            'AMap.Geocoder' // 正向地理编码
          ]
        }).then(AMap => {
          Vue.prototype.$AMap = AMap
          window.AMap = AMap
        }).catch(e => {
          throw (new Error('加载高德地图失败：' + e))
        })
      }
      
      export default {
        install
      }
      ```

      

   6. 在main.js中导入

      ```javascript
      import AMap from './utils/amap'
      Vue.use(AMap)
      ```

      

   7. 在页面MapContainer.vue 中使用：围栏

      ```vue
      <template>
        <div class="draw-container">
          <div id="container" ref="Container" />
          <el-card :body-style="{ padding: '2px' }">
            <div class="draw-vector">
              <div>
                <el-button :disabled="isNewCircle" size="small" type="text" @click="addCircleHandle">新建围栏</el-button>
              </div>
              <div>
                <el-button size="small" type="text" @click="editCircle">开始编辑</el-button>
              </div>
              <div>
                <el-button size="small" type="text" @click="finishCircle">结束编辑</el-button>
              </div>
            </div>
          </el-card>
          <!-- 新建围栏 -->
          <el-dialog title="新建围栏" :visible.sync="addDialog" width="25%">
            <el-form :model="addFenceObj" label-width="120px">
              <el-form-item label="所属区域">
                <el-select v-model="addFenceObj.cityName" style="width: 100%;" placeholder="请选择所属区域">
                  <el-option v-for="item in areaOptions" :key="item.code" :label="item.name" :value="item.code" />
                </el-select>
              </el-form-item>
              <el-form-item label="围栏名称">
                <el-input v-model="addFenceObj.name" />
              </el-form-item>
              <el-form-item label="半径">
                <el-input v-model="addFenceObj.radius" />
              </el-form-item>
              <el-form-item label="形状">
                <el-select v-model="addFenceObj.type" style="width: 100%;" placeholder="请选择形状">
                  <el-option v-for="item in shapeOptions" :key="item.type" :label="item.name" :value="item.type" />
                </el-select>
              </el-form-item>
              <el-form-item label="经纬度">
                <el-input v-model="addFenceObj.center" />
              </el-form-item>
              <el-form-item label="位置">
                <el-input v-model="addFenceObj.location" />
              </el-form-item>
              <el-form-item label="描述信息">
                <el-input v-model="addFenceObj.description" />
              </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
              <el-button size="small" @click="addDialog = false">取 消</el-button>
              <el-button size="small" type="primary" @click="submitFence">确 定</el-button>
            </div>
          </el-dialog>
        </div>
      </template>
      
      <script>
      import {
        createGeoFence,
        reviseGeoFence
      } from '@/api/fence'
      import IconEdit from '@/assets/amap/icon-edit.png'
      import IconInfo from '@/assets/amap/icon-info.png'
      
      export default {
        name: 'VectorDrawing',
        props: {
          fenceInfo: {
            type: Object,
            default() {
              return {}
            }
          }
        },
        data() {
          return {
            MAP: null, // 实例化地图
            circleObj: null,
            markerObj: null,
            circleEditor: null,
            infoWindow: null,
            mouseTool: null, // 鼠标工具
            isNewCircle: false, // 是否可以新建围栏 false:是 true:否
            toolStatus: '', // 工具状态
            /* 新建围栏 */
            addDialog: false,
            areaOptions: [],
            shapeOptions: [
              {
                type: 0,
                name: '圆形'
              }
            ],
            addFenceObj: {
              center: '', // 经纬度
              cityCode: '', // 城市编码
              cityName: '', // 城市名称
              countryCode: '100000', // 国家编码
              countryName: '中国', // 国家名称
              description: '', // 描述
              location: '', // 经纬度 所在位置详情
              name: '', // 围栏名称
              provinceCode: '', // 省编码
              provinceName: '', // 省名称
              radius: '', // 半径
              type: 0 // 0:圆形 1:矩形 2:多边形
            }
          }
        },
        watch: {
          fenceInfo(value) {
            console.log(value)
            this.circleObj = new this.$AMap.Circle({
              center: [...value.center.split(',').map(value => parseFloat(value))],
              radius: value.radius,
              borderWeight: 3,
              strokeColor: '#409EFF',
              strokeWeight: 4,
              strokeOpacity: 0.6,
              fillOpacity: 0.1,
              strokeStyle: 'solid',
              strokeDasharray: [10, 10],
              fillColor: '#67C23A',
              zIndex: 50
            })
            this.markerObj = new this.$AMap.Marker({
              position: [...value.center.split(',').map(value => parseFloat(value))],
              icon: IconInfo,
              offset: new this.$AMap.Pixel(-20, -50),
              zIndex: 80
            })
            this.markerObj.setMap(this.MAP)
            value.area = (Math.PI * Math.pow(value.radius, 2)).toFixed(2)
            this.markerObj.on('click', event => {
              this.openInfoWindow(value)
            })
      
            this.circleObj.setMap(this.MAP)
            // 缩放地图到合适的视野级别
            this.MAP.setFitView([this.circleObj])
          }
        },
        mounted() {
          this.$nextTick(() => {
            this.initMap()
          })
        },
        methods: {
          initMap() {
            setTimeout(() => {
              // 使用 $refs 获取节点
              this.MAP = new this.$AMap.Map(this.$refs.Container, {
                zoom: 13, // 级别
                center: [114.05, 22.55],
                viewMode: '2D' // 地图模式
              })
              // 在图面添加工具条控件，工具条控件集成了缩放、平移、定位等功能按钮在内的组合控件
              this.MAP.addControl(new this.$AMap.ToolBar({ position: 'RT' }))
              // 在图面添加比例尺控件，展示地图在当前层级和纬度下的比例尺
              this.MAP.addControl(new this.$AMap.Scale())
      
              this.mouseTool = new this.$AMap.MouseTool(this.MAP)
              // 监听draw事件可获取画好的覆盖物
              this.mouseTool.on('draw', e => {
                this.circleObj = e.obj
                this.circleEditor = new this.$AMap.CircleEditor(this.MAP, this.circleObj)
                this.circleEditor.open()
                this.mouseTool.close()
                this.circleEditor.on('move', e => {
                  this.areaOptions = []
                  this.addFenceObj.center = `${e.lnglat.lng},${e.lnglat.lat}`
                  this.getAddressByLngLat([e.lnglat.lng, e.lnglat.lat])
                })
                this.circleEditor.on('adjust', e => {
                  console.log(e)
                  this.areaOptions = []
                  this.addFenceObj.radius = e.radius
                  this.addFenceObj.center = `${e.lnglat.lng},${e.lnglat.lat}`
                  this.getAddressByLngLat([e.lnglat.lng, e.lnglat.lat])
                })
                this.circleEditor.on('end', event => {
                  this.areaOptions = []
                  this.addFenceObj.radius = event.target.getRadius()
                  this.addFenceObj.center = `${event.target.getCenter()}`
                  const fenceCenter = this.addFenceObj.center.split(',').map(value => parseFloat(value))
                  this.getAddressByLngLat([...fenceCenter])
                })
              })
            }, 1000)
          },
          /* 提交围栏 */
          submitFence() {
            if (this.addFenceObj.id) {
              reviseGeoFence(this.addFenceObj).then(res => {
                this.$message({
                  type: 'success',
                  message: '创建成功!'
                })
                this.addDialog = false
                Object.assign(this.$data.addFenceObj, this.$options.data().addFenceObj)
                this.$nextTick(function() {
                  this.initMap()
                })
                this.$emit('updateFence')
              })
            } else {
              createGeoFence(this.addFenceObj).then(res => {
                this.$message({
                  type: 'success',
                  message: '创建成功!'
                })
                this.addDialog = false
                Object.assign(this.$data.addFenceObj, this.$options.data().addFenceObj)
                this.$nextTick(function() {
                  this.initMap()
                })
                this.$emit('updateFence')
              })
            }
          },
          /* 新建围栏 */
          addCircleHandle() {
            this.isNewCircle = true
            this.drawCover('circle')
          },
          /* 编辑围栏 */
          editCircle() {
            console.log('编辑')
          },
          /* 结束编辑围栏 */
          finishCircle() {
            this.addDialog = true
            this.circleEditor.close()
          },
          /* 根据经纬度查询位置 */
          getAddressByLngLat(lngLat) {
            const geocoder = new this.$AMap.Geocoder({
              city: '全国'
            })
            geocoder.getAddress(lngLat, (status, result) => {
              if (status === 'complete' && result.regeocode) {
                const cityCode = result.regeocode.addressComponent.citycode
                const cityName = result.regeocode.addressComponent.city ? result.regeocode.addressComponent.city : result.regeocode.addressComponent.province
                this.addFenceObj.provinceCode = result.regeocode.addressComponent.citycode.slice(0, 2)
                this.addFenceObj.provinceName = result.regeocode.addressComponent.province
                this.addFenceObj.location = result.regeocode.formattedAddress
                this.addFenceObj.cityName = cityName
                this.addFenceObj.cityCode = cityCode
                this.areaOptions.push({
                  code: cityCode,
                  name: cityName
                })
              } else {
                this.$message.info('根据经纬度查询地址失败')
              }
            })
          },
          /* 信息窗体 */
          openInfoWindow(info) {
            const infoList = ['<div class="info-container">']
            infoList.push(`<div style="color: #303133;font-size: 16px">创新大厦</div>`)
            infoList.push(`<div style="margin: 10px 0;border: 2px solid #ebeef5;padding: 10px;border-radius: 4px;color: #303133;font-size: 14px">`)
            infoList.push(`<div>面积：${info.area} m²</div>`)
            infoList.push(`<div>编号：${info.id}</div>`)
            infoList.push(`<div>类型：圆形</div>`)
            infoList.push(`<div>经度：${info.center.split(',')[1]}</div>`)
            infoList.push(`<div>纬度：${info.center.split(',')[0]}</div>`)
            infoList.push(`<div>位置：${info.location}</div>`)
            infoList.push(`<div>备注：${info.description}</div>`)
            infoList.push(` </div>`)
            infoList.push(`<img src="${IconEdit}" alt="编辑" width="20px" style="cursor: pointer" onclick="openEditHandle()">`)
            infoList.push(`</div>`)
            this.infoWindow = new this.$AMap.InfoWindow({
              content: infoList.join(''),
              offset: new this.$AMap.Pixel(-10, -50)
            })
            this.infoWindow.open(this.MAP, [...info.center.split(',').map(value => parseFloat(value))])
            window.openEditHandle = () => {
              this.addFenceObj = { ...info }
              console.log(this.addFenceObj)
              this.addDialog = true
            }
          },
          /* 鼠标绘制覆盖物 */
          drawCover(type) {
            switch (type) {
              case 'marker': {
                this.mouseTool.marker({
                  // 同Marker的Option设置
                })
                break
              }
              case 'polyline': {
                this.mouseTool.polyline({
                  strokeColor: '#80d8ff'
                  // 同Polyline的Option设置
                })
                break
              }
              case 'polygon': {
                this.mouseTool.polygon({
                  fillColor: '#00b0ff',
                  strokeColor: '#80d8ff'
                  // 同Polygon的Option设置
                })
                break
              }
              case 'rectangle': {
                this.mouseTool.rectangle({
                  fillColor: '#00b0ff',
                  strokeColor: '#80d8ff'
                  // 同Polygon的Option设置
                })
                break
              }
              case 'circle': {
                this.mouseTool.circle({
                  borderWeight: 3,
                  strokeColor: '#409EFF',
                  strokeWeight: 4,
                  strokeOpacity: 0.6,
                  fillOpacity: 0.1,
                  strokeStyle: 'solid',
                  strokeDasharray: [10, 10],
                  fillColor: '#67C23A',
                  zIndex: 50
                })
                break
              }
            }
          }
        }
      }
      </script>
      
      <style lang="scss" scoped>
      @import "//at.alicdn.com/t/font_3215613_p8w68cx24kf.css";
      .draw-container {
        position: relative;
        width: 100%;
        height: 100%;
      }
      #container {
        width: 100%;
        height: 100%;
      }
      .el-card {
        position: absolute;
        top: 15px;
        right: 50px;
      }
      .draw-vector {
        width: 300px;
        display: flex;
        align-items: center;
        justify-content: space-between;
        color: #797979;
        font-size: 14px;
        text-align: center;
        div {
          flex: 1;
          cursor: pointer;
          border-right: 1px dashed #797979;
          &:last-child {
            border-right: none;
          }
        }
      }
      </style>
      
      ```

      

   8. 在页面MapContainer.vue 中使用：路线

      ```vue
      <template>
        <div class="page-content">
          <el-form :inline="true" :model="searchObj" size="small">
            <el-form-item label="路线名称">
              <el-input v-model="searchObj.name" clearable placeholder="输入路线名称" />
            </el-form-item>
            <el-form-item label="始发地">
              <el-input v-model="searchObj.origin" clearable placeholder="输入始发地" />
            </el-form-item>
            <el-form-item label="目的地">
              <el-input v-model="searchObj.destination" clearable placeholder="输入目的地" />
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="submitHandle">查询</el-button>
              <el-button @click="resetHandle">重置</el-button>
              <el-button type="primary" @click="createHandle">新增</el-button>
            </el-form-item>
          </el-form>
          <el-table v-loading="courseLoading" :data="courseList" border stripe size="small">
            <el-table-column label="序号" type="index" width="50" align="center" />
            <el-table-column prop="name" label="班线名称" align="center" />
            <el-table-column prop="id" label="班线编码" align="center" />
            <el-table-column prop="origin" show-overflow-tooltip label="始发地" align="center" />
            <el-table-column prop="destination" show-overflow-tooltip label="目的地" align="center" />
            <el-table-column prop="hairStatus" label="晚发报警" align="center" />
            <el-table-column prop="arrivalStatus" label="晚到报警" align="center" />
            <el-table-column prop="roadOffsetStatus" label="道路偏移报警" align="center" />
            <el-table-column label="操作" align="center">
              <template slot-scope="scope">
                <el-button type="text" size="small" @click="editHandle(scope.row)">编辑</el-button>
                <el-button type="text" size="small" @click="alarmConfigHandle(scope.row)">报警配置</el-button>
                <el-popconfirm
                  title="确定删除吗？"
                  @confirm="deleteHandle(scope.row)"
                >
                  <el-button slot="reference" type="text" size="small">删除</el-button>
                </el-popconfirm>
                <el-button type="text" size="small" @click="examineHandle(scope.row)">查看</el-button>
              </template>
            </el-table-column>
          </el-table>
          <el-pagination
            :current-page="pageInfo.pageNo"
            :page-size="pageInfo.pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="pageInfo.total"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
          />
          <!-- 创建 -->
          <el-drawer
            ref="routeDrawer"
            title="新增路线"
            :visible.sync="isRouteDialog"
            direction="rtl"
            size="70%"
          >
            <el-row>
              <el-col :span="12">
                <el-card shadow="never">
                  <div slot="header" class="header">
                    <span class="header-title">围栏列表</span>
                    <el-input size="small" placeholder="请输入内容" style="width: 250px">
                      <el-button slot="append" icon="el-icon-search" @click="searchFenceHandle" />
                    </el-input>
                  </div>
                  <div class="fence">
                    <div v-for="(item,index) in fenceList" :key="index" class="fence-item" :class="fenceCheckList.find(value => value.id === item.id)?'active':''" @click="fenceHandle(index, item)">
                      <div>围栏名称：<span>{{ item.name }}</span></div>
                      <el-tooltip :content="item.location" placement="bottom-start">
                        <div class="address">地址：<span>{{ item.location }}</span></div>
                      </el-tooltip>
                    </div>
                  </div>
                </el-card>
              </el-col>
              <el-col :span="12">
                <el-card shadow="never">
                  <div slot="header" class="header">
                    <span class="header-title">已选围栏</span>
                  </div>
                  <div class="over">
                    <div v-for="(item,index) in fenceCheckList" :key="index" class="over-item">
                      <div class="serial">{{ index+1 }}</div>
                      <div class="over-name">{{ item.name }}</div>
                      <i class="el-icon-delete" @click="fenceClearHandle(item)" />
                    </div>
                  </div>
                </el-card>
              </el-col>
            </el-row>
            <el-card shadow="never" class="map-contain" :body-style="{padding: 0}">
              <div id="container" ref="Container" />
              <el-form ref="routeRef" :model="routeObj" size="small" :rules="routeRules" label-width="100px">
                <div class="route-planning">
                  <div>路线规划</div>
                  <el-button size="small" :disabled="fenceCheckLen<2" icon="el-icon-set-up" type="primary" @click="planningHandle">智能规划</el-button>
                </div>
                <el-form-item label="总里程(km)" prop="mileage">
                  <el-input v-model="routeObj.mileage" placeholder="输入总里程" />
                </el-form-item>
                <el-form-item label="运输时长(h)" prop="drivelTime">
                  <el-input v-model="routeObj.drivelTime" placeholder="输入运输时长" />
                </el-form-item>
                <el-form-item label="路线名称" prop="name">
                  <el-input v-model="routeObj.name" placeholder="输入路线名称" />
                </el-form-item>
                <el-form-item label="描述" prop="description">
                  <el-input v-model="routeObj.description" placeholder="输入描述" />
                </el-form-item>
                <el-form-item>
                  <el-button @click="cancelRoute">取 消</el-button>
                  <el-button type="primary" :loading="routeLoading" @click="createSubmitHandle">{{ routeLoading ? '提交中 ...' : '确 定' }}</el-button>
                </el-form-item>
              </el-form>
            </el-card>
          </el-drawer>
          <!-- 报警配置 -->
          <el-dialog title="报警配置" :visible.sync="alarmConfigDialog" width="30%">
            <el-form :model="alarmConfigObj" :inline="true" label-width="120px">
              <el-form-item label="晚发预警">
                <el-switch v-model="alarmConfigObj.lateHairWarning" :active-value="1" :inactive-value="0" />
              </el-form-item>
              <el-form-item label="晚发报警">
                <el-switch v-model="alarmConfigObj.lateHairPolice" :active-value="1" :inactive-value="0" />
              </el-form-item>
              <el-form-item label="晚到预警">
                <el-switch v-model="alarmConfigObj.lateArrivalWarning" :active-value="1" :inactive-value="0" />
              </el-form-item>
              <el-form-item label="晚到报警">
                <el-switch v-model="alarmConfigObj.lateArrivalPolice" :active-value="1" :inactive-value="0" />
              </el-form-item>
              <el-form-item label="道路偏移报警">
                <el-switch v-model="alarmConfigObj.roadDeviationAlarm" :active-value="1" :inactive-value="0" />
              </el-form-item>
            </el-form>
            <div slot="footer">
              <el-button size="small" @click="cancelConfigHandle">取 消</el-button>
              <el-button size="small" type="primary" @click="submitConfigHandle">保 存</el-button>
            </div>
          </el-dialog>
      
        </div>
      </template>
      
      <script>
      import { createClassLine,
        deleteClassLine,
        geoFenceInfo,
        getClassLine,
        reviseClassLine
      } from '@/api/fence'
      import IconLocation from '@/assets/amap/icon-location.png'
      
      export default {
        name: 'Courses',
        data() {
          return {
            MAP: null, // 实例化地图
            courseLoading: false,
            courseList: [],
            searchObj: {
              name: '', // 班线名称
              origin: '', // 始发地
              destination: '' // 目的地
            },
            pageInfo: {
              pageNo: 1,
              pageSize: 10,
              total: 0
            },
            /* 围栏列表 */
            fenceList: [],
            fenceCheckList: [], // 选中围栏
            /* 新增路线 */
            isRouteDialog: false,
            routeLoading: false,
            routeObj: {
              classLineGeofenceOrderDtos: [], // 选中围栏列表
              destinationGeofenceId: '', // 目的围栏ID
              drivelTime: '', // 运输时长
              mileage: '', // 总里程
              name: '', // 路线名称
              originGeofenceId: '', // 起点围栏ID
              description: '' // 描述
            },
            routeRules: {
              mileage: { required: true, message: '总里程不能为空', trigger: 'blur' },
              drivelTime: { required: true, message: '运输时长不能为空', trigger: 'blur' },
              name: { required: true, message: '路线名称不能为空', trigger: 'blur' },
              description: { required: true, message: '描述不能为空', trigger: 'blur' }
            },
            /* 报警配置 */
            alarmConfigDialog: false,
            alarmConfigObj: {
              lateHairWarning: 0, // 晚发预警
              lateHairPolice: 0, // 晚发报警
              lateArrivalWarning: 0, // 晚到预警
              lateArrivalPolice: 0, // 晚到报警
              roadDeviationAlarm: 0 // 道路偏移报警
            }
          }
        },
        computed: {
          fenceCheckLen() {
            return this.fenceCheckList.length
          }
        },
        mounted() {
          this.getClassLineList()
          this.geoFenceInfo()
        },
        methods: {
          /* 实例化地图 */
          initMap() {
            setTimeout(() => {
              // 使用 $refs 获取节点
              this.MAP = new this.$AMap.Map(this.$refs.Container, {
                zoom: 14, // 级别
                resizeEnable: true,
                center: [114.057939, 22.543527] // 中心点坐标
              })
              // 在图面添加工具条控件，工具条控件集成了缩放、平移、定位等功能按钮在内的组合控件
              this.MAP.addControl(new this.$AMap.ToolBar({ position: 'LT' }))
              // 在图面添加比例尺控件，展示地图在当前层级和纬度下的比例尺
              this.MAP.addControl(new this.$AMap.Scale())
            })
          },
          /* 获取围栏列表 */
          geoFenceInfo() {
            geoFenceInfo().then(res => {
              this.fenceList = this.treeToArray(res).filter(item => item.id)
              for (const item of this.fenceList) {
                item.checkIndex = false
              }
            })
          },
          /* 获取货运订单 */
          getClassLineList() {
            this.courseLoading = true
            getClassLine({ ...this.searchObj, ...this.pageInfo }).then(res => {
              this.pageInfo.total = parseInt(res.totalCount)
              this.courseList = res.list
            }).finally(() => {
              this.courseLoading = false
            })
          },
          /* 查询 */
          submitHandle() {
            this.pageInfo.pageNo = 1
            this.getClassLineList()
          },
          /* 重置 */
          resetHandle() {
            Object.assign(this.$data.searchObj, this.$options.data().searchObj)
          },
          /* 查询围栏列表 */
          searchFenceHandle() {
            this.geoFenceInfo()
          },
          /* 创建 */
          createHandle() {
            Object.assign(this.$data.routeObj, this.$options.data().routeObj)
            this.fenceCheckList = []
            this.$nextTick(() => {
              this.initMap()
            })
            this.isRouteDialog = true
          },
          /* 编辑 */
          editHandle(row) {
            this.fenceCheckList = []
            for (const item of row.classLineGeofenceOrderVos) {
              for (const fence of this.fenceList) {
                if (item.geofenceId === fence.id) {
                  fence.checkIndex = true
                  this.fenceCheckList.push(fence)
                }
              }
            }
            this.routeObj.id = row.id
            this.routeObj.drivelTime = row.drivelTime
            this.routeObj.mileage = row.mileage
            this.routeObj.name = row.name
            this.routeObj.description = row.description
            this.$nextTick(() => {
              this.initMap()
            })
            this.isRouteDialog = true
          },
          /* 报警配置 */
          alarmConfigHandle(row) {
            this.alarmConfigDialog = true
          },
          /* 删除 */
          deleteHandle(row) {
            deleteClassLine(row.id).then(res => {
              this.$message({
                type: 'success',
                message: '删除成功!'
              })
              this.getClassLineList()
            })
          },
          /* 查看 */
          examineHandle(row) {
            console.log('查看')
          },
          /* 提交新增货单 */
          createSubmitHandle() {
            this.routeObj.classLineGeofenceOrderDtos = []
            if (this.fenceCheckList.length < 2) {
              this.$message({
                message: '请选择围栏（2个及以上）',
                type: 'warning'
              })
              return
            }
            for (let i = 0; i < this.fenceCheckList.length; i++) {
              this.routeObj.classLineGeofenceOrderDtos.push({
                geofenceId: this.fenceCheckList[i].id, // 围栏ID
                orderNumber: i // 排序序号
              })
            }
            this.routeObj.originGeofenceId = this.fenceCheckList[0].id
            this.routeObj.destinationGeofenceId = this.fenceCheckList[this.fenceCheckList.length - 1].id
            this.$refs['routeRef'].validate((valid) => {
              if (valid) {
                if (this.routeLoading) {
                  return
                }
                this.$confirm('确定要提交表单吗？').then(_ => {
                  this.routeLoading = true
                  if (this.routeObj.id) {
                    reviseClassLine(this.routeObj).then(res => {
                      this.$message({
                        type: 'success',
                        message: '修改成功'
                      })
                      this.isRouteDialog = false
                      this.getClassLineList()
                    }).finally(() => {
                      this.routeLoading = false
                    })
                  } else {
                    createClassLine(this.routeObj).then(res => {
                      this.$message({
                        type: 'success',
                        message: '创建成功'
                      })
                      this.isRouteDialog = false
                      this.getClassLineList()
                    }).finally(() => {
                      this.routeLoading = false
                    })
                  }
                })
              }
            })
          },
          /* 取消新增 */
          cancelRoute() {
            this.$refs['routeRef'].resetFields()
            this.routeLoading = false
            this.isRouteDialog = false
            clearTimeout(this.timer)
          },
          /* 围栏列表多选 */
          fenceHandle(i, item) {
            // 创建点覆盖物
            const position = item.center.split(',').map(value => parseFloat(value))
            const marker = new this.$AMap.Marker({
              position: new this.$AMap.LngLat(...position),
              icon: IconLocation,
              offset: new this.$AMap.Pixel(-13, -30)
            })
            this.MAP.add(marker)
            this.MAP.setFitView()
            const idx = this.fenceCheckList.find(value => value.id === item.id)
            if (idx) {
              item.checkIndex = false
              this.fenceCheckList = this.fenceCheckList.filter(value => value.id !== idx.id)
            } else {
              item.checkIndex = true
              this.fenceCheckList.push(item)
            }
          },
          /* 取消已选 */
          fenceClearHandle(item) {
            console.log(item)
            this.fenceCheckList = this.fenceCheckList.filter(value => value.id !== item.id)
          },
          // 智能规划
          planningHandle() {
            // 构造路线导航类
            const driving = new this.$AMap.Driving({
              map: this.MAP,
              policy: this.$AMap.DrivingPolicy.LEAST_TIME, // 最快捷模式
              ferry: 1 // 是否可以使用轮渡
            })
            // 根据起终点经纬度规划驾车导航路线
            const startPoint = this.fenceCheckList[0].center.split(',').map(value => parseFloat(value))
            const endPoint = this.fenceCheckList[this.fenceCheckList.length - 1].center.split(',').map(value => parseFloat(value))
            const wayPoint = this.fenceCheckList.slice(1, this.fenceCheckList.length - 1).map(value => value.center.split(',').map(item => parseFloat(item)))
            const wayPoints = wayPoint.map(value => new this.$AMap.LngLat(...value))
            driving.search(new this.$AMap.LngLat(...startPoint), new this.$AMap.LngLat(...endPoint), {
              waypoints: wayPoints
            }, (status, result) => {
              if (status === 'complete') {
                if (result.routes && result.routes.length) {
                  // 绘制第一条路线，也可以按需求绘制其它几条路线
                  console.log('绘制驾车路线完成')
                  this.routeObj.mileage = result.routes[0].distance / 1000
                  this.routeObj.drivelTime = parseFloat(this.secondToDate(result.routes[0].time))
                }
              } else {
                console.log(`获取驾车数据失败：${result}`)
              }
            })
          },
          /* 提交报警配置 */
          submitConfigHandle() {
            this.alarmConfigDialog = true
          },
          /* 取消报警配置 */
          cancelConfigHandle() {
            Object.assign(this.$data.alarmConfigObj, this.$options.data().alarmConfigObj)
            this.alarmConfigDialog = true
          },
          handleSizeChange(val) {
            this.pageInfo.pageSize = val
            this.getClassLineList()
          },
          handleCurrentChange(val) {
            this.pageInfo.pageNo = val
            this.getClassLineList()
          },
          // 秒转化成 时分秒
          secondToDate(result) {
            const iDays = result / 60 / 60
            return iDays.toFixed(2)
          },
          // 树结构扁平化
          treeToArray(tree) {
            return tree.reduce((res, item) => {
              const { children, ...i } = item
              return res.concat(i, children && children.length ? this.treeToArray(children) : [])
            }, [])
          }
        }
      }
      </script>
      
      <style lang="scss" scoped>
      .page-content {
        margin: 30px;
      }
      
      .el-form, .el-pagination {
        margin-top: 30px;
      }
      
      .el-pagination {
        float: right;
      }
      #container {
        width: 100%;
        height: calc(100vh - 450px);
      }
      .el-row {
        width: 100%;
      }
      .header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        &-title {
          height: 32px;
          line-height: 32px;
        }
      }
      .fence {
        display: flex;
        flex-wrap: wrap;
        height: 250px;
        overflow-y: auto;
        align-content: flex-start;
        &-item {
          position: relative;
          width: 200px;
          height: 60px;
          border: 1px solid #409EFF;
          border-radius: 5px;
          cursor: pointer;
          margin: 5px;
          padding: 5px;
          display: flex;
          flex-direction: column;
          justify-content: center;
          color: #666;
          font-size: 14px;
          span {
            color: #343434;
          }
          &.active {
            background-color: #409EFF;
            span {
              color: #FFFFFF;
            }
          }
          .address {
            margin-top: 6px;
            width: 195px;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
          }
        }
      }
      .over {
        display: flex;
        flex-wrap: wrap;
        height: 250px;
        overflow-y: auto;
        align-content: flex-start;
        &-item {
          position: relative;
          width: 200px;
          height: 60px;
          border: 1px solid #409EFF;
          border-radius: 5px;
          cursor: pointer;
          margin: 5px;
          color: #343434;
          font-size: 16px;
          .el-icon-delete {
            display: none;
          }
          .serial {
            width: 100%;
            height: 24px;
            background: #409EFF;
            line-height: 24px;
            color: #fff;
            border-radius: 5px 5px 0 0;
            text-align: center;
          }
          &:hover {
            i {
              display: block;
              position: absolute;
              right: 2px;
              top: 2px;
              padding: 2px;
              z-index: 10;
              cursor: pointer;
              color: #FFFFFF;
            }
          }
        }
        &-name {
          height: 36px;
          line-height: 36px;
          text-align: center;
          font-size: 18px;
        }
      }
      .map-contain {
        position: relative;
        .el-form {
          padding: 15px 10px;
          background-color: #FFFFFF;
          position: absolute;
          top: -25px;
          right: 5px;
          bottom: 5px;
          width: 400px;
          .route-planning {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 15px;
          }
        }
      }
      ::v-deep .amap-logo {
        display: none;
        opacity: 0 !important;
      }
      ::v-deep .amap-copyright {
        opacity: 0;
      }
      </style>
      
      ```

      

   9. 
