#### 效果图：

![image-20230225142826140](https://raw.githubusercontent.com/chnjames/cloudImg/main/blog/202302251428782.png)

#### 实现代码：
```vue
<template>
    <el-form class="form" ref="form" :model="form" :rules="rules" label-width="80px">
            <el-form-item label="设备名称" prop="equipmentName">
              <el-select v-model="form.equipmentName" placeholder="请选择设备名称" clearable style="width: 100%">
                <el-option v-for="(item, index) in menuOptions" :key="index" :label="item.name" :value="item.name"/>
              </el-select>
            </el-form-item>
            <el-form-item label="模板类型" prop="templateType">
              <el-select v-model="form.templateType" placeholder="请选择模板类型" clearable style="width: 100%">
                <el-option v-for="(item, index) in menuOptions" :key="index" :label="item.name" :value="item.name"/>
              </el-select>
            </el-form-item>
            <el-card shadow="never" v-for="(item, index) in form.partsList" :key="index">
              <el-row :gutter="10" type="flex" justify="space-between">
                <el-col :span="23">
                  <el-form-item label="设备部位" :prop="'partsList.'+ index + '.location'" :rules="partsListRules.location">
                    <el-input v-model="item.location" placeholder="请输入设备部位"/>
                  </el-form-item>
                  <el-form-item label="部位图片" :prop="'partsList.' + index + '.image'" :rules="partsListRules.image">
                    <imageUpload v-model="item.image" :limit="1" :is-show-tip="false"/>
                  </el-form-item>
                  <el-card shadow="never" v-for="(task, idx) in item.taskList" :key="idx">
                    <el-row :gutter="10" type="flex" justify="space-between">
                      <el-col :span="23">
                        <el-form-item label="任务名称" :prop="'partsList.' + index + '.taskList.' + idx + '.taskName'" :rules="taskListRules.taskName">
                          <el-input v-model="task.taskName" placeholder="请输入任务名称"/>
                        </el-form-item>
                        <el-form-item label="执行结果" required>
                          <el-row :gutter="20" type="flex" justify="space-between">
                            <el-col :span="8">
                              <el-form-item :prop="'partsList.' + index + '.taskList.' + idx + '.outcome'" :rules="taskListRules.outcome">
                                <el-select v-model="task.outcome" style="width: 100%" placeholder="请选择执行结果" clearable>
                                  <el-option v-for="(item, index) in menuOptions" :key="index" :label="item.name" :value="item.name"/>
                                </el-select>
                              </el-form-item>
                            </el-col>
                            <el-col :span="4">
                              <el-form-item :prop="'partsList.' + index + '.taskList.' + idx + '.termOne'" :rules="taskListRules.termOne">
                                <el-select v-model="task.termOne" style="width: 100%" placeholder="请选择" clearable>
                                  <el-option v-for="(item, index) in menuOptions" :key="index" :label="item.name" :value="item.name"/>
                                </el-select>
                              </el-form-item>
                            </el-col>
                            <el-col :span="4">
                              <el-form-item :prop="'partsList.' + index + '.taskList.' + idx + '.valueOne'" :rules="taskListRules.valueOne">
                                <el-input v-model="task.valueOne" placeholder="数值"/>
                              </el-form-item>
                            </el-col>
                            <el-col :span="4">
                              <el-form-item :prop="'partsList.' + index + '.taskList.' + idx + '.termTwo'" :rules="taskListRules.termTwo">
                                <el-select v-model="task.termTwo" style="width: 100%" placeholder="请选择" clearable>
                                  <el-option v-for="(item, index) in menuOptions" :key="index" :label="item.name" :value="item.name"/>
                                </el-select>
                              </el-form-item>
                            </el-col>
                            <el-col :span="4">
                              <el-form-item :prop="'partsList.' + index + '.taskList.' + idx + '.valueTwo'" :rules="taskListRules.valueTwo">
                                <el-input v-model="task.valueTwo" placeholder="数值"/>
                              </el-form-item>
                            </el-col>
                          </el-row>
                        </el-form-item>
                        <el-row :gutter="20">
                          <el-col :span="12">
                            <el-form-item label="选择备件" :prop="'partsList.' + index + '.taskList.' + idx + '.spare'" :rules="taskListRules.spare">
                              <el-select v-model="task.spare" style="width: 100%" placeholder="请选择" clearable>
                                <el-option v-for="(item, index) in menuOptions" :key="index" :label="item.name" :value="item.name"/>
                              </el-select>
                            </el-form-item>
                          </el-col>
                          <el-col :span="12">
                            <el-form-item label="备件数量" :prop="'partsList.' + index + '.taskList.' + idx + '.quantity'" :rules="taskListRules.quantity">
                              <el-input-number v-model="task.quantity" :min="0" />
                            </el-form-item>
                          </el-col>
                        </el-row>
                      </el-col>
                      <i class="inter el-icon-delete"  v-if="item.taskList.length > 1" @click="deleteInterItems(task, idx)"></i>
                    </el-row>
                  </el-card>
                  <el-button type="primary" icon="el-icon-plus" @click="addInterItems(item, index)" plain style="margin-top: 10px;width: 100%">添加任务项</el-button>
                </el-col>
                <i class="outer el-icon-delete" v-if="form.partsList.length > 1" @click="deleteOuterItems(item, index)"></i>
              </el-row>
            </el-card>
            <!--添加外层内容-->
            <el-button type="primary" plain icon="el-icon-plus" @click="addOuterItems" style="margin-top: 10px;width: 100%">添加部位 & 任务项</el-button>
          </el-form>
</template>
<script>
export default {
  name: 'Template',
  components: {
    ImageUpload
  },
  data() {
    return {
      // 表单参数
      form: {
        equipmentName: undefined, // 设备名称
        templateType: undefined, // 模板类型
        partsList: [
          {
            location: undefined, // 设备部位
            image: undefined, // 图片
            taskList: [
              {
                taskName: undefined, // 任务名称
                outcome: undefined, // 执行结果
                termOne: undefined, // 条件1
                valueOne: undefined, // 条件1的值
                termTwo: undefined, // 条件2
                valueTwo: undefined, // 条件2的值
                spare: undefined, // 备件
                previous: undefined, // 前值
                after: undefined, // 后值
                quantity: 0 // 备件数量
              }
            ]
          }
        ]
      },
      // 表单校验
      rules: {
        equipmentName: [
          { required: true, message: '设备名称不能为空', trigger: 'blur' },
          { max: 30, message: '设备名称不能超过30个字', trigger: 'blur' }
        ],
        templateType: [
          { required: true, message: '模板类型不能为空', trigger: 'blur' },
          { max: 30, message: '模板类型不能超过30个字', trigger: 'blur' }
        ]
      },
      partsListRules: {
        location: [
          { required: true, message: '设备部位不能为空', trigger: 'blur' },
          { max: 30, message: '设备部位不能超过30个字', trigger: 'blur' }
        ],
        image: [
          { required: true, message: '图片不能为空', trigger: 'blur' }
        ]
      },
      taskListRules: {
        taskName: [
          { required: true, message: '任务名称不能为空', trigger: 'blur' },
          { max: 30, message: '任务名称不能超过30个字', trigger: 'blur' }
        ],
        outcome: [
          { required: true, message: '执行结果不能为空', trigger: 'blur' },
          { max: 30, message: '执行结果不能超过30个字', trigger: 'blur' }
        ],
        termOne: [
          { required: true, message: '条件1不能为空', trigger: 'blur' },
          { max: 30, message: '条件1不能超过30个字', trigger: 'blur' }
        ],
        valueOne: [
          { required: true, message: '条件1的值不能为空', trigger: 'blur' },
          { max: 30, message: '条件1的值不能超过30个字', trigger: 'blur' }
        ],
        termTwo: [
          { required: true, message: '条件2不能为空', trigger: 'blur' },
          { max: 30, message: '条件2不能超过30个字', trigger: 'blur' }
        ],
        valueTwo: [
          { required: true, message: '条件2的值不能为空', trigger: 'blur' },
          { max: 30, message: '条件2的值不能超过30个字', trigger: 'blur' }
        ],
        spare: [
          { required: true, message: '备件不能为空', trigger: 'blur' },
          { max: 30, message: '备件不能超过30个字', trigger: 'blur' }
        ],
        previous: [
          { required: true, message: '前值不能为空', trigger: 'blur' },
          { max: 30, message: '前值不能超过30个字', trigger: 'blur' }
        ],
        after: [
          { required: true, message: '后值不能为空', trigger: 'blur' },
          { max: 30, message: '后值不能超过30个字', trigger: 'blur' }
        ],
        quantity: [
          { required: true, message: '备件数量不能为空', trigger: 'blur' }
        ]
      },
    }
  },
  methods: {
      /** 添加任务项操作 */
    addInterItems(item, index) {
      console.log(item, index)
      item.taskList.push({
        taskName: undefined, // 任务名称
        outcome: undefined, // 执行结果
        termOne: undefined, // 条件1
        valueOne: undefined, // 条件1的值
        termTwo: undefined, // 条件2
        valueTwo: undefined, // 条件2的值
        spare: undefined, // 备件
        previous: undefined, // 前值
        after: undefined, // 后值
        quantity: 0 // 备件数量
      })
    },
    /** 删除任务项操作 */
    deleteInterItems(task, idx) {
      console.log(task, idx)
      this.form.partsList[idx].taskList.splice(task, 1)
    },
    /** 添加部位&任务项操作 */
    addOuterItems() {
      this.form.partsList.push({
        location: undefined,
        image: undefined,
        taskList: [
          {
            taskName: undefined, // 任务名称
            outcome: undefined, // 执行结果
            termOne: undefined, // 条件1
            valueOne: undefined, // 条件1的值
            termTwo: undefined, // 条件2
            valueTwo: undefined, // 条件2的值
            spare: undefined, // 备件
            previous: undefined, // 前值
            after: undefined, // 后值
            quantity: 0 // 备件数量
          }
        ]
      })
    },
    /** 删除部位&任务项操作 */
    deleteOuterItems(item, index) {
      console.log(item, index)
      this.form.partsList.splice(index, 1)
    },
  }
}
</script>
```

