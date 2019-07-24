# multi-cascader-base-element
基于 element-ui 封装的多选级联下拉框 （新版element已有该component，适用于旧版）
> UI 风格与 element 完全一致，支持半选、远程搜索等。
> 无需 install ，拿来即用。

![](https://img-blog.csdnimg.cn/20190724163748426.gif)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724154925165.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3E5NTU0ODg1NA==,size_16,color_FFFFFF,t_70)

## How To Use
- copy multi-cascader-base-element/ 文件夹下所有内容至自己项目
- 在需要使用的 component 中引入：
```
import elCascaderMulti from '@/components/multi-cascader-base-element/multi-cascader.vue'
```
- 在 components 中注册：
```
components: {
  elCascaderMulti
}
```
- 使用：（详细配置见 Document）
```
<el-cascader-multi v-model="checkList" :data="data" valueKey="id"> </el-cascader-multi>

// script
data() {
  return {
    data: [
    {
      id: 'zhinan',
      label: '指南',
      children: [{
        id: 'shejiyuanze',
        label: '设计原则',
        children: [{
          id: 'yizhi',
          label: '一致'
        }, {
          id: 'fankui',
          label: '反馈'
        }, {
          id: 'xiaolv',
          label: '效率'
        }, {
          id: 'kekong',
          label: '可控'
        }]
      }, {
        id: 'daohang',
        label: '导航',
        children: [{
          id: 'cexiangdaohang',
          label: '侧向导航'
        }, {
          id: 'dingbudaohang',
          label: '顶部导航'
        }]
      }]
    }, {
      id: 'zujian',
      label: '组件',
      children: [{
        id: 'basic',
        label: 'Basic',
        children: [{
          id: 'layout',
          label: 'Layout 布局'
        }, {
          id: 'color',
          label: 'Color 色彩'
        }]
      }, {
        id: 'form',
        label: 'Form',
        children: [{
          id: 'radio',
          label: 'Radio 单选框'
        }, {
          id: 'checkbox',
          label: 'Checkbox 多选框'
        }]
      }, {
        id: 'data',
        label: 'Data',
        children: [{
          id: 'table',
          label: 'Table 表格'
        }, {
          id: 'tag',
          label: 'Tag 标签'
        }]
      }, {
        id: 'notice',
        label: 'Notice',
        children: [{
          id: 'alert',
          label: 'Alert 警告'
        }, {
          id: 'loading',
          label: 'Loading 加载'
        }]
      }, {
        id: 'navigation',
        label: 'Navigation',
        children: [{
          id: 'menu',
          label: 'NavMenu 导航菜单'
        }]
      }, {
        id: 'others',
        label: 'Others',
        children: [{
          id: 'dialog',
          label: 'Dialog 对话框'
        }, {
          id: 'tooltip',
          label: 'Tooltip 文字提示'
        }]
      }]
    }, {
      id: 'ziyuan',
      label: '资源',
      children: [{
        id: 'jiaohu',
        label: '组件交互文档'
      }]
    }],
    checkList: ['jiaohu', 'yizhi']
  };
},
```

## Document

### Attributes

参数 | 说明 | 类型 | 可选值 | 默认值
---|---|---|---|---
data | 必传, 列表数据 | Array | -- | --
onlyShowChecked | 是否只显示选中的 | Boolean | -- | false
isShowIndeterminate | 是否显示半选 | Boolean | -- | true
v-model | 必传, 默认值和返回值 | Array | -- | --
separator | 分隔符(onlyShowChecked为false时有效) | String | '/', '-'等 | '/'
filterable | 是否可搜索 | Boolean | -- | false
filterMethod | 自定义搜索方法 | Function | -- | --
popperClass | Select 下拉框的类名 | String | -- | --
reserveKeyword | 多选且可搜索时，是否在选中一个选项后保留当前的搜索关键词 | Boolean | -- | true
valueKey | value 唯一标识的键名 | String | -- | 'value'
labelKey | label 唯一标识的键名 | String | -- | 'label'
childrenKey | childrenKey 唯一标识的键名 | String | -- | 'children'
expandTrigger | 下级展开方式 | String | 'click', 'hover' | 'click'

> **补充说明**

- v-model值：即页面刷新默认值，也是 checked 之后的返回数组（只需传选中的id，eg：['jiaohu', 'yizhi']）注意不要传重复，传了父节点就不要传子节点（错误案例：['zhinan', 'yizhi']）
- valueKey：即 item 的唯一标识，eg: item 的 id
- labelKey：即 item 的展示名， eg: item 的 name
- childrenKey：即 item 下级的 key

> **依赖**：Element-UI、Vue、lodash
