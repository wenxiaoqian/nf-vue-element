<template>
  <div class="home">
    <el-row :gutter="20">
      <!--不循环row，直接循环col，放不下会自动往下串行。-->
      <el-col :span="13">
        <div style="padding-left:20px;">
          列数：<el-slider v-model="meta.formColCount" :min="1" :max="5"></el-slider>
          <elForm v-model="model" v-model:partModel="partModel" :meta="meta"/>
        </div>
      </el-col>
      <el-col :span="3">
        完整的 model 值：<br><br>
        <template v-for="(item, key) in model" :key="key">
          {{key}}：{{item}}<br>
        </template>
        <br>
        <hr>
        相应的 model 值：<br><br>
        <template v-for="(item, key) in partModel" :key="key">
          {{key}}：{{item}}<br>
        </template>
      </el-col>
      <el-col :span="8">
        <!--排序-->
        <div @dragover="allowDrop($event)">
          <div :style="styleCol"
            v-for="(id, index) in meta.colOrder"
            :key="index"
            draggable="true"
            @dragstart="myDrag(id, index)"
          >
            {{meta.itemMeta[id].label}}
            <span style="font-size:9px;color:blue;" @drop="myDrop1(id, index, $event)">
              插入
            </span>
            <span style="font-size:9px;color:blue;" @drop="myDrop2(id, index, $event)">
              交换
            </span>
            <!--设置一个组件占多少格子-->
            <elselect
              @input="spanChange"
              style="width:90px;"
              v-model="meta.itemMeta[id].colCount"
              :meta="spanSelectMeta"
            />
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { inject, reactive } from 'vue'
import { getControlTypeOptionList } from '@/components/controlConfig/config.js'
import elFormConfig from '@/components/nf-el-form-item/el-form-map.js'
import elForm from '@/components/nf-el-form/el-form-div'
// @ is an alias to /src

// 设置组件占位的meta
const spanSelectMeta = {
  controlId: 9527,
  colName: 'controlType',
  controlType: 160,
  defaultValue: '1',
  optionList: [
    { value: 1, label: '一格' },
    { value: 2, label: '二格' },
    { value: 3, label: '三格' },
    { value: 4, label: '四格' },
    { value: -2, label: '一半' },
    { value: -3, label: '三份' },
    { value: -4, label: '四份' },
    { value: -5, label: '五份' }
  ],
  isClear: false,
  disabled: false,
  title: '占位',
  placeholder: '选择占位'
}

// 拖拽排序的框
const styleCol = {
  width: '250px',
  height: '26px',
  margin: '10px',
  padding: '10px',
  border: '1px solid #aaaaaa'
}

// 拖拽排序管理类
const dragManage = (meta) => {
  // 记录拖拽组件的数据
  const ctl = reactive({
    id: 1, // 组件ID
    index: 1 // 在数组里的下标
  })

  // 允许拖拽，拦截默认事件
  const allowDrop = (ev) => {
    ev.preventDefault()
  }

  // 选择的字段
  const myDrag = (id, index) => {
    ctl.id = id
    ctl.index = index
  }

  // 小号插入大号的前面
  const myDrop1 = (id, index, e) => {
    if (ctl.index < index - 1) {
      for (let i = ctl.index; i < index; i++) {
        meta.colOrder[i] = meta.colOrder[i + 1]
      }
      meta.colOrder[index - 1] = ctl.id
    }
    meta.reload = !meta.reload
  }
  // 交互顺序
  const myDrop2 = (id, index, e) => {
    meta.colOrder[ctl.index] = id
    meta.colOrder[index] = ctl.id
    meta.reload = !meta.reload
  }

  return {
    allowDrop,
    myDrag,
    myDrop1,
    myDrop2
  }
}

export default {
  name: 'eleBase',
  components: {
    elForm,
    ...elFormConfig.formItemList
  },
  setup () {
    const userOnline = inject('userOnline')
    console.log('注入测试：', userOnline)
    userOnline.isOnline = true

    const model = reactive({})
    const partModel = reactive({})
    const { typeOptionList } = getControlTypeOptionList()

    // 读取json，获得配置。
    const json = require('@/views/json/meta-form.json')
    const meta = reactive(json.formMeta) // 表单的meta
    // 添加重新绑定的开关
    meta.reload = false
    // 设置组件类型
    meta.itemMeta[103].optionList = typeOptionList
    console.log('meta', meta)

    // 实现拖拽排序的功能
    const {
      allowDrop,
      myDrag,
      myDrop1,
      myDrop2
    } = dragManage(meta)

    // 重新渲染表单
    const spanChange = () => {
      meta.reload = !meta.reload
    }

    return {
      model,
      partModel,
      meta,
      // 排序
      allowDrop,
      // 字段排序
      styleCol,
      myDrag,
      myDrop1,
      myDrop2,
      spanChange, // 重新渲染表单
      spanSelectMeta
    }
  }

}
</script>
