### 页面


```vue
<template>
  <div class="app-container">
    <el-form :inline="true" size="small" :model="scenesObj">
      <el-form-item label="规则名称">
        <el-input v-model="scenesObj.name" placeholder="输入规则名称" />
      </el-form-item>
      <el-form-item label="状态">
        <el-select v-model="scenesObj.status" placeholder="选择状态">
          <el-option label="启用" :value="0" />
          <el-option label="停用" :value="1" />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="searchHandle">查询</el-button>
        <el-button @click="resetHandle">重置</el-button>
        <el-button type="primary" @click="addHandle">新增</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="scenesLoading" :data="scenesList" stripe border size="small">
      <el-table-column type="index" width="50" align="center" />
      <el-table-column prop="name" label="规则名称" align="center" />
      <el-table-column prop="remark" label="规则描述" align="center" />
      <el-table-column label="状态" align="center">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.enableState" :active-value="0" :inactive-value="1" @change="stateHandle(scope.row, $event)" />
        </template>
      </el-table-column>
      <el-table-column prop="updateAt" label="创建时间" align="center" />
      <el-table-column label="操作" width="300" align="center">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="editHandle(scope.row)">编辑</el-button>
          <el-button type="text" size="small" style="display: none" @click="triggerHandle(scope.row)">触发</el-button>
          <el-button type="text" size="small">日志</el-button>
          <el-popconfirm
            title="确定删除吗？"
            style="margin: 0 10px"
            @confirm="deleteHandle(scope.row)"
          >
            <el-button slot="reference" type="text" size="small" :disabled="scope.row.enableState === 0">删除</el-button>
          </el-popconfirm>
          <el-button type="text" size="small">查看</el-button>
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
    <el-drawer title="场景编辑" :visible.sync="isScenesDrawer" size="55%">
      <el-form ref="scenesInfoRef" size="small" class="scenes-form" :model="scenesInfoObj" :rules="scenesInfoRules" label-width="100px">
        <el-row>
          <el-col :span="12">
            <el-form-item label="规则名称" prop="name">
              <el-input v-model="scenesInfoObj.name" placeholder="输入规则名称" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="状态" prop="enableState">
              <el-radio-group v-model="scenesInfoObj.enableState">
                <el-radio :label="0">启用</el-radio>
                <el-radio :label="1">禁用</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="规则描述">
              <el-input v-model="scenesInfoObj.remark" type="textarea" :rows="3" :maxlength="200" placeholder="输入规则描述" />
            </el-form-item>
          </el-col>
        </el-row>
        <h4>场景联动规则</h4>
        <div class="form-container">
          <div class="label">
            <div>
              触发器（Trigger）
              <el-tooltip content="触发器不能为空，消息只要满足触发条件中任意一个即可触发。" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </div>
            <el-button type="primary" icon="el-icon-plus" size="mini" @click="addTrigger">新增触发器</el-button>
          </div>
          <article>
            <section v-for="(trigger, index) in triggerArr" :key="index" class="item">
              <div class="item-form">
                <div class="item-form-title">触发器{{ index + 1 }}</div>
                <div class="item-form-option">
                  <el-select v-model="trigger.conTriggerType" placeholder="请选择触发器类型" size="small">
                    <el-option v-for="item in conTriggerList" :key="item.type" :label="item.name" :value="item.type" />
                  </el-select>
                  <el-select v-show="trigger.isConTriggerType" v-model="trigger.productKey" placeholder="请选择产品" size="small" @change="productChange($event, trigger)">
                    <el-option v-for="item in productsList" :key="item.productId" :label="item.name" :value="item.productId" />
                  </el-select>
                  <el-select v-show="trigger.isConTriggerType" v-model="trigger.deviceKey" placeholder="请选择设备" size="small">
                    <el-option v-for="item in trigger.devicesList" :key="item.deviceId" :label="item.name" :value="item.deviceId" />
                  </el-select>
                  <br>
                  <el-select v-show="trigger.isTriggerType" v-model="trigger.triggerType" placeholder="请选择触发方式" size="small" @change="triggerTypeChange($event, trigger)">
                    <el-option v-for="item in sceneTriggerTypeList" :key="item.type" :label="item.name" :value="item.type" />
                  </el-select>
                  <el-select v-show="trigger.isLiveCycle" v-model="trigger.liveCycle" placeholder="请选择上下线" size="small">
                    <el-option v-for="item in liveCycleList" :key="item.type" :label="item.name" :value="item.type" />
                  </el-select>
                  <el-select v-show="trigger.isEvent" v-model="trigger.event" placeholder="请选择事件" size="small">
                    <el-option v-for="item in trigger.productEventList" :key="item.identifier" :label="item.name" :value="item.identifier" />
                  </el-select>
                  <el-select v-show="trigger.isProductKey" v-model="trigger.compareAttr" placeholder="请选择属性" size="small">
                    <el-option v-for="item in trigger.attributesList" :key="item.identifier" :label="item.name" :value="item.identifier" />
                  </el-select>
                  <el-select v-show="trigger.isProductKey" v-model="trigger.compareMethod" placeholder="请选择比较模式" size="small">
                    <el-option v-for="item in compareMethodList" :key="item.type" :label="item.name" :value="item.type" />
                  </el-select>
                  <el-input v-show="trigger.isProductKey" v-model="trigger.compareValue" placeholder="请输入比较值" size="small" />
                </div>
              </div>
              <el-button :disabled="triggerArr.length <= 1" type="text" @click="deleteTrigger(trigger)">删除</el-button>
            </section>
          </article>
        </div>
        <div class="form-container">
          <div class="label">
            <div>
              执行条件（Condition）
              <el-tooltip content="执行条件可为空，消息需要满足所有执行条件才会被处理。" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </div>
            <el-button type="primary" icon="el-icon-plus" size="mini" @click="addCondition">新增执行条件</el-button>
          </div>
          <article>
            <section v-for="(condition, index) in conditionArr" :key="index" class="item">
              <div class="item-form">
                <div class="item-form-title">执行条件{{ index + 1 }}</div>
                <div class="item-form-option">
                  <el-select v-model="condition.conditionType" placeholder="请选择执行条件" size="small" @change="conditionTypeChange($event, condition)">
                    <el-option v-for="item in sceneConditionTypeList" :key="item.type" :label="item.name" :value="item.type" />
                  </el-select>
                  <el-select v-show="condition.isConConditionType" v-model="condition.productKey" placeholder="请选择产品" size="small" @change="conditionProductChange($event, condition)">
                    <el-option v-for="item in productsList" :key="item.productId" :label="item.name" :value="item.productId" />
                  </el-select>
                  <el-select v-show="condition.isConConditionType" v-model="condition.deviceKey" placeholder="请选择设备" size="small">
                    <el-option v-for="item in condition.devicesList" :key="item.deviceId" :label="item.name" :value="item.deviceId" />
                  </el-select>
                  <br>
                  <el-select v-show="condition.isProductKey" v-model="condition.compareAttr" placeholder="请选择属性" size="small">
                    <el-option v-for="item in condition.attributesList" :key="item.identifier" :label="item.name" :value="item.identifier" />
                  </el-select>
                  <el-select v-show="condition.isProductKey" v-model="condition.compareMethod" placeholder="请选择比较模式" size="small">
                    <el-option v-for="item in compareMethodList" :key="item.type" :label="item.name" :value="item.type" />
                  </el-select>
                  <el-input v-show="condition.isProductKey" v-model="condition.compareValue" placeholder="请输入比较值" size="small" />
                </div>
              </div>
              <el-button type="text" @click="deleteCondition(condition)">删除</el-button>
            </section>
          </article>
        </div>
        <div class="form-container">
          <div class="label">
            <div>
              执行动作（Action）
              <el-tooltip content="执行动作不能为空，可设置多个动作，某一动作执行失败，不影响其他动作。" placement="top">
                <i class="el-icon-question" />
              </el-tooltip>
            </div>
            <el-button :disabled="actionArr.length >= 1" type="primary" icon="el-icon-plus" size="mini" @click="addAction">新增执行动作</el-button>
          </div>
          <article>
            <section v-for="(action, index) in actionArr" :key="index" class="item">
              <div class="item-form">
                <div class="item-form-title">执行动作{{ index + 1 }}</div>
                <div class="item-form-option">
                  <el-select v-model="action.actionType" placeholder="请选择执行动作" size="small">
                    <el-option v-for="item in sceneActionTypeList" :key="item.type" :label="item.name" :value="item.type" />
                  </el-select>
                </div>
              </div>
              <el-button :disabled="actionArr.length <= 1" type="text" @click="deleteAction(action)">删除</el-button>
            </section>
          </article>
        </div>
        <el-form-item>
          <el-button type="primary" @click="submitScenesHandle('scenesInfoRef')">确 定</el-button>
          <el-button @click="resetScenesHandle('scenesInfoRef')">取 消</el-button>
        </el-form-item>
      </el-form>
    </el-drawer>
  </div>
</template>

<script>
import {
  createScenes,
  deleteScenes,
  deviceList,
  enableScenes,
  productInfo,
  productList,
  scenesList,
  triggerScenes,
  updateScenes
} from '@/api/scenes'
import {
  COMPAREMETHODLIST,
  CONTRIGGERLIST,
  LIVECYCLELIST,
  SCENEACTIONTYPELIST,
  SCENECONDITIONTYPELIST,
  SCENETRIGGERTYPELIST
} from '@/views/config-manage/constants/manual-constants'

export default {
  name: 'SceneRules',
  data() {
    return {
      sceneActionTypeList: SCENEACTIONTYPELIST, // 场景执行动作类型
      sceneConditionTypeList: SCENECONDITIONTYPELIST, // 场景执行条件类型
      compareMethodList: COMPAREMETHODLIST, // 比较模式
      sceneTriggerTypeList: SCENETRIGGERTYPELIST, // 场景触发类型
      conTriggerList: CONTRIGGERLIST, // 选择触发器类型
      liveCycleList: LIVECYCLELIST, // 选择设备上下线
      productsList: [], // 产品列表
      scenesLoading: false,
      scenesObj: {
        status: '',
        name: ''
      },
      pageInfo: {
        pageNo: 1,
        pageSize: 10,
        total: 0
      },
      scenesList: [],
      /* 编辑新增场景规则 */
      sceneMaxObj: null,
      editStatusId: '', // 是否编辑状态ID
      isScenesDrawer: false,
      scenesInfoObj: {
        name: '',
        enableState: 0,
        remark: ''
      },
      scenesInfoRules: {
        name: [
          { required: true, message: '请输入活动名称', trigger: 'blur' },
          { min: 1, max: 50, message: '长度在 1 到 50 个字符', trigger: 'blur' }
        ],
        enableState: { required: true, message: '请选择状态', trigger: 'change' }
      },
      /* 新增场景规则 */
      triggerArr: [{
        conTriggerType: '',
        productKey: '',
        deviceKey: '',
        liveCycle: '',
        triggerType: '',
        compareAttr: '',
        compareMethod: '',
        compareValue: '',
        event: '',
        isLiveCycle: false,
        isProductKey: false,
        isConTriggerType: false,
        isTriggerType: false,
        isEvent: false,
        devicesList: [],
        attributesList: [], // 属性列表
        productEventList: [], // 事件列表
        attributesObj: null // 属性遥测对象
      }],
      conditionArr: [{
        conditionType: '',
        productKey: '',
        deviceKey: '',
        compareAttr: '',
        compareMethod: '',
        compareValue: '',
        isConConditionType: false,
        isProductKey: false,
        devicesList: [],
        attributesList: [], // 属性列表
        attributesObj: null // 属性遥测对象
      }],
      actionArr: [{
        actionType: ''
      }]
    }
  },
  watch: {
    triggerArr: {
      handler(arr) {
        for (const item of arr) {
          if (item.conTriggerType !== '') {
            item.isConTriggerType = true
          }
          if (item.productKey !== '') {
            item.isTriggerType = true
          }
          if (item.triggerType !== '') {
            switch (item.triggerType) {
              case 1:
                item.isLiveCycle = false
                item.isProductKey = true
                item.isEvent = false
                item.attributesList = item.attributesObj && item.attributesObj.copyAttributesList
                break
              case 2:
                item.isLiveCycle = false
                item.isProductKey = true
                item.isEvent = false
                item.attributesList = item.attributesObj && item.attributesObj.copyTelemetryList
                break
              case 3:
                item.isLiveCycle = false
                item.isProductKey = false
                item.isEvent = true
                break
              case 4:
                item.isLiveCycle = true
                item.isProductKey = false
                item.isEvent = false
                break
            }
          }
        }
        console.log('watch监听trigger', arr)
      },
      immediate: true,
      deep: true
    },
    conditionArr: {
      handler(arr) {
        for (const item of arr) {
          if (item.conditionType !== '') {
            item.isConConditionType = true
          }
          if (item.productKey !== '') {
            item.isProductKey = true
          }
          if (item.conditionType !== '') {
            switch (item.conditionType) {
              case 0:
                item.attributesList = item.attributesObj && item.attributesObj.copyAttributesList
                break
              case 1:
                item.attributesList = item.attributesObj && item.attributesObj.copyTelemetryList
                break
            }
          }
        }
        console.log('watch监听condition', arr)
      },
      immediate: true,
      deep: true
    }
  },
  mounted() {
    this.getScenesList()
    this.productList()
  },
  methods: {
    /* 获取场景列表*/
    getScenesList() {
      this.scenesList = []
      this.scenesLoading = true
      scenesList({ ...this.scenesObj, ...this.pageInfo }).then(res => {
        this.pageInfo.total = parseInt(res.totalCount)
        this.scenesList = res.list
      }).finally(() => {
        this.scenesLoading = false
      })
    },
    /* 获取产品列表 */
    productList() {
      this.productsList = []
      productList().then(res => {
        this.productsList = res
      })
    },
    /* 创建场景规则 */
    createScenes(scene) {
      createScenes(scene).then(res => {
        this.$message({
          type: 'success',
          message: '创建成功'
        })
        this.isScenesDrawer = false
        this.getScenesList()
      })
    },
    /* 更新场景规则 */
    updateScenes(scene) {
      updateScenes(scene).then(res => {
        this.$message({
          type: 'success',
          message: '更新成功'
        })
        this.isScenesDrawer = false
        this.getScenesList()
      })
    },
    // 触发器选择产品
    productChange(val, trigger) {
      trigger.deviceKey = ''
      trigger.compareAttr = ''
      deviceList({ productId: val, pageNo: 1, pageSize: 100 }).then(res => {
        trigger.devicesList = res.list
      })
      productInfo({ productId: val }).then(res => {
        trigger.attributesObj = {
          copyAttributesList: res.productAttributes,
          copyTelemetryList: res.productTelemetrys
        }
        trigger.productEventList = res.productEventVos
      })
      console.log(trigger.attributesObj)
    },
    // 切换触发方式
    triggerTypeChange(val, trigger) {
      trigger.compareAttr = ''
    },
    // 新增触发器
    addTrigger() {
      this.triggerArr.push({
        conTriggerType: '',
        productKey: '',
        deviceKey: '',
        liveCycle: '',
        triggerType: '',
        compareAttr: '',
        compareMethod: '',
        compareValue: '',
        event: '',
        isLiveCycle: false,
        isProductKey: false,
        isConTriggerType: false,
        isTriggerType: false,
        isEvent: false,
        devicesList: []
      })
    },
    // 删除触发器
    deleteTrigger(trigger) {
      const index = this.triggerArr.indexOf(trigger)
      if (index !== -1) {
        this.triggerArr.splice(index, 1)
      }
    },
    // 新增执行条件
    addCondition() {
      this.conditionArr.push({
        conditionType: '',
        productKey: '',
        deviceKey: '',
        compareAttr: '',
        compareMethod: '',
        compareValue: '',
        isConConditionType: false,
        isProductKey: false,
        devicesList: []
      })
    },
    // 执行条件选择产品
    conditionProductChange(val, condition) {
      condition.deviceKey = ''
      condition.compareAttr = ''
      deviceList({ productId: val, pageNo: 1, pageSize: 100 }).then(res => {
        condition.devicesList = res.list
      })
      productInfo({ productId: val }).then(res => {
        console.log(res)
        condition.attributesObj = {
          copyAttributesList: res.productAttributes,
          copyTelemetryList: res.productTelemetrys
        }
      })
    },
    // 切换执行条件
    conditionTypeChange(val, condition) {
      condition.compareAttr = ''
    },
    // 删除执行条件
    deleteCondition(condition) {
      const index = this.conditionArr.indexOf(condition)
      if (index !== -1) {
        this.conditionArr.splice(index, 1)
      }
    },
    // 新增执行动作
    addAction() {
      this.actionArr.push({
        actionType: ''
      })
    },
    // 删除执行动作
    deleteAction(action) {
      const index = this.actionArr.indexOf(action)
      if (index !== -1) {
        this.actionArr.splice(index, 1)
      }
    },
    /* 删除规则 */
    deleteScenes(id) {
      deleteScenes(id).then(res => {
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        this.getScenesList()
      })
    },
    /* 启用停用规则 */
    enableScenes(id, status) {
      enableScenes(id, status).then(res => {
        if (status === 0) {
          this.$message({
            type: 'success',
            message: '启用成功!'
          })
        } else {
          this.$message({
            type: 'success',
            message: '停用成功!'
          })
        }
      }).catch(() => {
        this.getScenesList()
      })
    },
    /* 触发场景 */
    triggerScenes(id) {
      triggerScenes(id).then(res => {
        this.$message({
          type: 'success',
          message: '触发成功!'
        })
      })
    },
    /* 查询 */
    searchHandle() {
      this.pageInfo.pageNo = 1
      this.getScenesList()
    },
    /* 重置 */
    resetHandle() {
      Object.assign(this.$data.scenesObj, this.$options.data().scenesObj)
      Object.assign(this.$data.pageInfo, this.$options.data().pageInfo)
      this.getScenesList()
    },
    /* 删除报警配置 */
    deleteHandle(row) {
      this.deleteScenes(row.id)
    },
    /* 修改状态 */
    stateHandle(row, val) {
      this.enableScenes(row.id, val)
    },
    /* 触发规则 */
    triggerHandle(row) {
      this.triggerScenes(row.id)
    },
    /* 新增 */
    addHandle() {
      this.sceneMaxObj = null
      this.editStatusId = ''
      Object.assign(this.$data.scenesInfoObj, this.$options.data().scenesInfoObj)
      // Object.assign(this.$data.triggerArr, this.$options.data().triggerArr)
      // Object.assign(this.$data.conditionArr, this.$options.data().conditionArr)
      // Object.assign(this.$data.actionArr, this.$options.data().actionArr)
      console.log(this.$data.triggerArr, this.$options.data().triggerArr)
      this.triggerArr = [{
        conTriggerType: '',
        productKey: '',
        deviceKey: '',
        liveCycle: '',
        triggerType: '',
        compareAttr: '',
        compareMethod: '',
        compareValue: '',
        event: '',
        isLiveCycle: false,
        isProductKey: false,
        isConTriggerType: false,
        isTriggerType: false,
        isEvent: false,
        devicesList: []
      }]
      this.conditionArr = [{
        conditionType: '',
        productKey: '',
        deviceKey: '',
        compareAttr: '',
        compareMethod: '',
        compareValue: '',
        isConConditionType: false,
        isProductKey: false,
        devicesList: []
      }]
      this.actionArr = [{
        actionType: ''
      }]
      this.isScenesDrawer = true
    },
    /* 编辑场景 */
    async editHandle(row) {
      this.editStatusId = row.id
      for (const item of row.triggers) {
        item.conTriggerType = 0
        await deviceList({ productId: item.productKey, pageNo: 1, pageSize: 100 }).then(res => {
          item.devicesList = res.list
        })
        await productInfo({ productId: item.productKey }).then(res => {
          item.attributesObj = {
            copyAttributesList: res.productAttributes,
            copyTelemetryList: res.productTelemetrys
          }
          item.productEventList = res.productEventVos
        })
      }
      for (const el of row.conditions) {
        await deviceList({ productId: el.productKey, pageNo: 1, pageSize: 100 }).then(res => {
          el.devicesList = res.list
        })
        await productInfo({ productId: el.productKey }).then(res => {
          el.attributesObj = {
            copyAttributesList: res.productAttributes,
            copyTelemetryList: res.productTelemetrys
          }
        })
      }
      this.actionArr = [...row.actions]
      this.conditionArr = [...row.conditions]
      this.triggerArr = [...row.triggers]
      this.scenesInfoObj = {
        name: row.name,
        enableState: row.enableState,
        remark: row.remark
      }
      this.isScenesDrawer = true
    },
    /* 提交场景规则 */
    submitScenesHandle(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          const actionData = this.actionArr.filter(value => value.actionType !== '')
          const conditionData = this.conditionArr.filter(value => value.conditionType !== '')
          const triggerData = this.triggerArr.filter(value => value.conTriggerType !== '')
          this.sceneMaxObj = {
            id: this.editStatusId,
            actions: actionData,
            conditions: conditionData,
            triggers: triggerData,
            ...this.scenesInfoObj
          }
          console.log(this.sceneMaxObj)
          if (this.editStatusId) {
            this.updateScenes(this.sceneMaxObj)
          } else {
            this.createScenes(this.sceneMaxObj)
          }
        }
      })
    },
    /* 取消场景规则 */
    resetScenesHandle(formName) {
      this.isScenesDrawer = false
      this.$refs[formName].resetFields()
    },
    handleSizeChange(val) {
      this.pageInfo.pageSize = val
      this.getScenesList()
    },
    handleCurrentChange(val) {
      this.pageInfo.pageNo = val
      this.getScenesList()
    }
  }
}
</script>

<style lang="scss" scoped>
.el-pagination {
  margin-top: 20px;
  float: right;
}
.scenes-form {
  padding: 20px;
}
.form-container {
  margin-bottom: 16px;
  color: #333;
  .label {
    width: 95%;
    margin-bottom: 4px;
    font-size: 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .item {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    &-form {
      width: 95%;
      background-color: #f5f5f6;
      padding: 16px 16px 8px;
      margin-bottom: 8px;
      min-height: 90px;
      box-sizing: border-box;
      &-title {
        font-size: 12px;
        color: #111;
        margin-bottom: 8px;
      }
      &-option {
        .el-select, .el-input {
          width: 200px;
          margin-right: 8px;
          margin-bottom: 8px;
        }
      }
    }
  }
}
</style>
```
