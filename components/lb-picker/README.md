# uni-app picker 选择器

插件市场里面的 picker 选择器不满足自己的需求，所以自己写了一个简单的 picker 选择器，可扩展、可自定义，一般满足日常需要。  
Github：[https://github.com/liub1934/uni-lb-picker](https://github.com/liub1934/uni-lb-picker)  
插件市场：[https://ext.dcloud.net.cn/plugin?id=1111](https://ext.dcloud.net.cn/plugin?id=1111)

## 兼容性

H5 + 各平台小程序（百度除外）  
安卓及 IOS 未测试（应该也能运行）
已知问题：支付宝小程序单选可能会出现不对齐的现象

## 功能

1、单选  
2、多级联动，非多级联动，理论支持任意级数  
3、省市区选择，基于多级联动  
4、自定义选择器头部确定取消按钮颜色及插槽支持  
5、选择器可视区自定义滚动个数  
6、自定义数据字段，满足不同人的需求  
7、自定义选择器样式  
8、单选及非联动选择支持扁平化的简单数据，如下形式：

```javascript
// 单选列表
list1: ['选项1', '选项2', '选项2'],
// 非联动选择列表
list2: [
  ['选项1', '选项2', '选项3'],
  ['选项11', '选项22', '选项33'],
  ['选项111', '选项222', '选项333']
]
```

## 引入插件

单独引入，在需要使用的页面上 import 引入即可

```html
<template>
  <view>
    <lb-picker></lb-picker>
  </view>
</template>

<script>
  import LbPicker from '@/components/lb-picker'
  export default {
    components: {
      LbPicker
    }
  }
</script>
```

全局引入，`main.js`中 import 引入并注册即可全局使用

```jsvascript
import LbPicker from '@/components/lb-picker'
Vue.component("lb-picker", LbPicker)
```

npm 安装引入：

```shell
npm install uni-lb-picker
```

```jsvascript
import LbPicker from 'uni-lb-picker'
```

## 选择器数据格式

### 单选

常规数据

```javascript
list: [
  {
    label: '选项1',
    value: '1'
  },
  {
    label: '选项2',
    value: '2'
  }
]
```

扁平化简单数据

```javascript
list: ['选项1', '选项2']
```

### 多级联动

```javascript
list: [
  {
    label: '选项1',
    value: '1',
    children: [
      {
        label: '选项1-1',
        value: '1-1',
        children: [
          {
            label: '选项1-1-1',
            value: '1-1-1'
          }
        ]
      }
    ]
  }
]
```

### 非联动选择

常规数据

```javascript
list: [
  [
    { label: '选项1', value: '1' },
    { label: '选项2', value: '2' },
    { label: '选项3', value: '3' }
  ],
  [
    { label: '选项11', value: '11' },
    { label: '选项22', value: '22' },
    { label: '选项33', value: '33' }
  ],
  [
    { label: '选项111', value: '111' },
    { label: '选项222', value: '222' },
    { label: '选项333', value: '333' }
  ]
]
```

扁平化简单数据

```javascript
list: [
  ['选项1', '选项2', '选项3'],
  ['选项11', '选项22', '选项33'],
  ['选项111', '选项222', '选项333']
]
```

## 调用显示选择器

通过`ref`形式手动调用`show`方法显示，隐藏同理调用`hide`

```text
<lb-picker ref="picker"></lb-picker>

this.$refs.picker.show() // 显示
this.$refs.picker.hide() // 隐藏
```

## 绑定值及设置默认值

支持 vue 中`v-model`写法绑定值，无需自己维护选中值的索引。

```javascript
<lb-picker v-model="value1"></lb-picker>
<lb-picker v-model="value2"></lb-picker>

data () {
  return {
    value1: '' // 单选
    value2: [] // 多列联动选择
  }
}
```

## 多个选择器

通过设置不同的`ref`，然后调用即可

```javascript
<lb-picker ref="picker1"></lb-picker>
<lb-picker ref="picker2"></lb-picker>

this.$refs.picker1.show() // picker1显示
this.$refs.picker2.show() // picker2显示
```

## 省市区选择

省市区选择是基于多列联动选择，数据来源：[https://github.com/modood/Administrative-divisions-of-China](https://github.com/modood/Administrative-divisions-of-China)，  
省市区文件位于`/pages/demos/area-data-min.js`，自行引入即可，可参考`demo3省市区选择`，  
也可使用自己已有的省市区数据，如果数据字段不一样，也可以自定义，参考下方自定义数据字段。

## 自定义数据字段

为了满足不同人的需求，插件支持自定义数据字段名称， 插件默认的数据字段如下形式：

```javascript
list: [
  {
    label: '选择1',
    value: 1,
    children: []
  },
  {
    label: '选择1',
    value: 1,
    children: []
  }
]
```

如果你的数据字段和上面不一样，如下形式：

```javascript
list: [
  {
    text: '选择1',
    id: 1,
    child: []
  },
  {
    text: '选择1',
    id: 1,
    child: []
  }
]
```

通过设置参数中的`props`即可，如下所示：

```javascript
<lb-picker :props="myProps"></lb-picker>

data () {
  return {
    myProps: {
      label: 'text',
      value: 'id',
      children: 'child'
    }
  }
}
```

## 插槽使用

选择器支持一些可自定义化的插槽，如选择器的取消和确定文字按钮，如果需要对其自定义处理的话，比如加个 icon 图标之类的，可使用插槽，使用方法如下：

```html
<lb-picker>
  <view slot="cancel-text">我是自定义取消</view>
  <view slot="confirm-text">我是自定义确定</view>
</lb-picker>
```

其他插槽见下。

## 参数及事件

### Props

| 参数                    | 说明                                                                                        | 类型                | 可选值                                                           | 默认值                                            |
| :---------------------- | :------------------------------------------------------------------------------------------ | :------------------ | :--------------------------------------------------------------- | :------------------------------------------------ |
| value/v-model           | 绑定值，联动选择为 Array 类型                                                               | String/Number/Array | -                                                                | -                                                 |
| mode                    | 选择器类型，支持单列，多列联动                                                              | String              | selector 单选/multiSelector 多级联动/unlinkedSelector 多级非联动 | selector                                          |
| list                    | 选择器数据(v1.0.7 单选及非联动多选支持扁平数据：['选项 1', '选项 2'])                       | Array               | -                                                                | -                                                 |
| level                   | 多列联动层级，仅 mode 为 multiSelector 有效                                                 | Number              | -                                                                | 2                                                 |
| props                   | 自定义数据字段                                                                              | Object              | -                                                                | {label:'label',value:'value',children:'children'} |
| cancel-text             | 取消文字                                                                                    | String              | -                                                                | 取消                                              |
| cancel-color            | 取消文字颜色                                                                                | String              | -                                                                | #999                                              |
| confirm-text            | 确定文字                                                                                    | String              | -                                                                | 确定                                              |
| confirm-color           | 确定文字颜色                                                                                | String              | -                                                                | #007aff                                           |
| empty-text              | (v1.0.7 新增)选择器列表为空的时候显示的文字                                                 | String              | -                                                                | 暂无数据                                          |
| empty-color             | (v1.0.7 新增)暂无数据文字颜色                                                               | String              | -                                                                | #999                                              |
| column-num              | 可视滚动区域内滚动个数，最好设置奇数值                                                      | Number              | -                                                                | 5                                                 |
| radius                  | 选择器顶部圆角，支持 rpx，如 radius="10rpx"                                                 | String              | -                                                                | -                                                 |
| ~~column-style~~        | ~~选择器默认样式(已弃用，见下方自定义样式说明)~~                                            | Object              | -                                                                | -                                                 |
| ~~active-column-style~~ | ~~选择器选中样式(已弃用，见下方自定义样式说明)~~                                            | Object              | -                                                                | -                                                 |
| loading                 | 选择器是否显示加载中，可使用 loading 插槽自定义加载效果                                     | Boolean             | -                                                                | -                                                 |
| mask-color              | 遮罩层颜色                                                                                  | String              | -                                                                | rgba(0, 0, 0, 0.4)                                |
| close-on-click-mask     | 点击遮罩层是否关闭选择器                                                                    | Boolean             | true/false                                                       | true                                              |
| ~~change-on-init~~      | ~~(v1.0.7 已弃用)初始化时是否触发 change 事件~~                                             | Boolean             | true/false                                                       | -                                                 |
| dataset                 | (v1.0.7 新增)可以向组件中传递任意的自定义的数据，在`confirm`或`change`事件中可以取到        | Object              | -                                                                | -                                                 |
| show-header             | (v1.0.8 新增)是否显示选择器头部                                                             | Boolean             | -                                                                | - true                                            |
| inline                  | (v1.0.8 新增)inline 模式，开启后默认显示选择器，无需点击弹出，可以配合`show-header`一起使用 | Boolean             | -                                                                | -                                                 |

### 方法

| 方法名 | 说明       | 参数 |
| :----- | :--------- | :--- |
| show   | 打开选择器 | -    |
| hide   | 关闭选择器 | -    |

### Events

| 事件名称 | 说明                                     | 回调参数                                                                                                                                                                                                                                                                                                                             |
| :------- | :--------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| show     | 选择器打开时触发                         | -                                                                                                                                                                                                                                                                                                                                    |
| hide     | 选择器隐藏时触发                         | -                                                                                                                                                                                                                                                                                                                                    |
| change   | 选择器滚动时触发，此时不会改变绑定的值   | `{ index, item, value, change }` `index`触发滚动后新的索引，单选时是具体的索引值，多列联动选择时为数组。`item`触发滚动后新的的完整内容，包括`label`、`value`等，单选时为对象，多列选择时为数组对象。`value`触发滚动后新的 value 值，单列选择时为具体值，多列联动选择时为数组。`change`触发事件的类型，详情参考下面的 change 事件备注 |
| confirm  | 点击选择器确定时触发，此时会改变绑定的值 | 同上`change`事件说明                                                                                                                                                                                                                                                                                                                 |
| cancel   | 点击选择器取消时触发                     | 同上`change`事件说明                                                                                                                                                                                                                                                                                                                 |

### `change` 事件备注

如果绑定的值是空的，`change`触发后里面的内容都是列表的第一项。  
`change`事件会在以下情况触发：

- 初始化
- 绑定值 value 变化
- 选择器 list 列表变化
- 滚动选择器

以上情况会在回调函数中都可以取到`change`变化的类型，对应上面的情况包括以下：

- `init`
- `value`
- `list`
- `scroll`

根据这些类型大家可以在`change`的时候按需处理自己的业务逻辑，比如一种常见的情况，有默认值的时候需要显示默认值的文字，此时可以`change`事件中判断`change`的类型是否是`init`，如果是的话可以取事件回调中的`item`进行显示绑定值对应的文字信息。

```javascript
handleChange (e) {
  if (e.change === 'init') {
    console.log(e.item.label) // 单选 选项1
    console.log(e.item.map(item => item.label).join('-')) // 多选 选项1-选项11
  }
}
```

### 插槽

| 插槽名        | 说明                |
| :------------ | :------------------ |
| cancel-text   | 选择器取消文字插槽  |
| action-center | 选择器顶部中间插槽  |
| confirm-text  | 选择器确定文字插槽  |
| loading       | 选择器 loading 插槽 |
| empty         | 选择器 空数据 插槽  |

### 选择器自定义样式

原先的`column-style`和`active-column-style`已弃用，如需修改默认样式及选中样式参考`demo9`

```css
<style lang="scss" scoped>
/deep/ .lb-picker {
  .lb-picker-column-label {
    color: #f0ad4e;
  }
  .lb-picker-column-active {
    .lb-picker-column-label {
      color: #007aff;
      font-weight: 700;
    }
  }
}
</style>
```

### 获取选中值的文字

`@confirm`事件中可以拿到：

单选：

```javascript
handleConfirm (e) {
  console.log(e.item.label) // 选项1
}
```

联动选择：

```javascript
handleConfirm (e) {
  console.log(e.item.map(item => item.label).join('-')) // 选项1-选项11
}
```

## Tips

微信小程序端，滚动时在 iOS 自带振动反馈，可在系统设置 -> 声音与触感 -> 系统触感反馈中关闭

## 其他

其他功能参考示例 Demo 代码。
