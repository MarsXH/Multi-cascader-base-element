<template>
  <div class="multi-cascader-base-element">
    <el-select
      v-model="selectedLabels"
      multiple
      :popper-class="innerPopperClass"
      @remove-tag="removeOne"
      :filterable="filterable"
      :filter-method="innerFilterMethod"
      :reserve-keyword="reserveKeyword"
      @change="changeLabel"
      v-bind="$attrs"
      @blur="handleBlur"
      @visible-change="visibleChange"
      @focus="handleFocus"
      @clear="handleClear"
      :allow-create="false"
    >
      <span slot="prefix" v-if="$slots.prefix">
        <slot name="prefix"></slot>
      </span>
      <template v-if="!isSearching">
        <el-option value="__root">
          <div class="ground" @click.stop>
            <render-list
              :list="root.childNodes"
              :level="1"
              :active-list="activeList"
              @handle-click="handleClick"
              @handle-check="handleCheck"
              :label-key="labelKey"
              :expand-trigger="expandTrigger"
            ></render-list>
            <template v-for="item in maxLevellist">
              <div
                :class="`floor-item floor-position-left-${item.id + 1}`"
                :key="item.id"
                v-if="item.rendered"
                v-show="activeList.length >= item.id"
              >
                <render-list
                  :list="showData[item.id]"
                  :level="item.id + 1"
                  :active-list="activeList"
                  @handle-click="handleClick"
                  @handle-check="handleCheck"
                  :label-key="labelKey"
                  :expand-trigger="expandTrigger"
                ></render-list>
              </div>
            </template>
          </div>
        </el-option>
      </template>
      <template v-if="isSearching">
        <el-option
          v-for="item in searchResult"
          :value="item.showLabel"
          :key="getKey(item)"
        >
          <div style="width:100%;height:100%" @click.stop="selectOne(item)">
            {{item.showLabel}}
          </div>
        </el-option>
      </template>
    </el-select>
  </div>
</template>

<script>
import TreeStore from './lib/Tree.js'
import renderList from './render-list.vue'
export default {
  name: 'el-cascader-multi',
  components: {
    renderList
  },
  props: {
    data: { // 列表数据
      type: Array,
      default: () => [],
      required: true
    },
    onlyShowChecked: { // 是否只显示选中的
      type: Boolean,
      default: false
    },
    isShowIndeterminate: { // 是否显示半选
      type: Boolean,
      default: true
    },
    value: { // v-model值，页面刷新默认值（只需传选中的id，eg：['jiaohu', 'yizhi']）注意不要传重复，传了父节点就不要传子节点（错误案例：['zhinan', 'yizhi']）
      type: Array
    },
    separator: { // 分隔符（onlyShowChecked为false时有效）
      type: String,
      default: '/'
    },
    filterable: { // 是否可搜索
      type: Boolean,
      default: false
    },
    filterMethod: { // 自定义搜索方法
      type: Function
    },
    popperClass: { // Select 下拉框的类名
      type: String,
      default: ''
    },
    reserveKeyword: { // 多选且可搜索时，是否在选中一个选项后保留当前的搜索关键词
      type: Boolean,
      default: true
    },
    valueKey: { // 作为 value 唯一标识的键名，绑定值为对象类型时必填
      type: String,
      default: 'value'
    },
    labelKey: { // 作为 label 唯一标识的键名，绑定值为对象类型时必填
      type: String,
      default: 'label'
    },
    childrenKey: { // 作为 children 唯一标识的键名，绑定值为对象类型时必填
      type: String,
      default: 'children'
    },
    expandTrigger: { // 下级展开方式
      type: String,
      default: 'click'
    },
    selectChildren: { // 是否允许父子联动
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      selectedLabels: [],
      selectedIds: [],
      selectedNodes: [],
      activeClass: 'floor-width-1',
      store: {},
      root: [],
      maxLevellist: [],
      showData: {},
      activeList: [],
      searchText: '',
      searchResult: []
    }
  },
  computed: {
    isSearching () {
      return !(this.searchText.trim() === '')
    },
    innerPopperClass () {
      return `${this.popperClass} multi-cascader ${this.isSearching ? '' : 'multi-cascader-style'} ${this.activeClass}`
    }
  },
  watch: {
    data: {
      deep: true,
      handler () {
        this.init()
      }
    },
    selectedNodes () {
      this.$emit('change', this.selectedNodes.map(o => o[this.valueKey]))
    }
  },
  methods: {
    visibleChange (v) {
      if (!v){
        this.searchText = ''
      }
      this.$emit('visible-change', v)
    },
    handleBlur (e) {
      this.searchText = ''
      this.$emit('blur', e)
    },
    handleFocus (e) {
      this.$emit('focus', e)
    },
    handleClear () {
      this.$emit('clear')
    },
    selectOne (item) {
      item.checked = !item.checked
      this.handleCheck(item)
    },
    changeLabel (v) {
      this.$emit('change', v)
    },
    innerFilterMethod (v) {
      this.searchText = v
      let tempResult = this.store.nodeList
      if (v.trim() !== '') {
        this.activeClass = ''
        if (typeof this.filterMethod === 'function') {
          this.searchResult = this.filterMethod(tempResult, v)
        } else {
          tempResult = tempResult.filter(o => o.isLeaf)
          tempResult = tempResult.filter(o => o.showLabel.includes(v))
          this.searchResult = tempResult
        }
      } else {
        this.activeClass= 'floor-width-1'
      }
    },
    getKey () {
      return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
        let r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8)
        return v.toString(16)
      })
    },
    handleClick(node, levelIndex, level){
      if (this.maxLevellist[level - 1]) {
        this.maxLevellist[level - 1].rendered = true
      }
      this.activeClass = `floor-width-${node.isLeaf ? level : level + 1}`
      let tempList = _.cloneDeep(this.activeList)
      if (level < tempList.length) {
        tempList.splice(level)
      }
      tempList[level - 1] = node.id
      this.showData[level] = node.childNodes
      this.activeList = tempList
    },
    isIndeterminate (level, nodeList) {
      if (level === 0) return
      nodeList.forEach(node => {
        let checkedlList = node.childNodes.filter(child => child.checked)
        if (node.level - level === -1) {
          node.indeterminate = false
          let indeterminatelist = node.childNodes.filter(child => child.indeterminate)
          let sumLength = checkedlList.length + indeterminatelist.length
          if (sumLength && checkedlList.length < node.childNodes.length) {
            node.indeterminate = true
            this.isIndeterminate(level - 1, this.root.childNodes)
          }
        } else if (node.level !== level) {
          this.isIndeterminate(level, node.childNodes)
        }
        if (checkedlList === node.childNodes.length) {
          node.indeterminate = false
        }
      })
    },
    handleCheck (node) {
      node.check(node.checked)
      this.selectedIds = this.store.selectedIds
      this.updateSelect(this.store.selectedIds)
      let result = this.selectedNodes.map(o => {
        if (!this.onlyShowChecked) {
          let level = o.level
          let valueKey = ''
          let node = _.cloneDeep(o)
          while (level !== 0) {
            valueKey = node[this.valueKey] + (valueKey ? this.separator : '') + valueKey
            node = node.parent
            level--
          }
          return valueKey
        }
        return o[this.valueKey]
      })
      this.$emit('input', result)
      if (this.isShowIndeterminate && this.selectChildren) this.isIndeterminate(node.level, this.root.childNodes)
    },
    removeOne (v) {
      let targetNode = _.find(this.selectedNodes, { showLabel: v })
      if (!this.onlyShowChecked) {
        let str = v.substring(v.lastIndexOf(this.separator) + 1)
        targetNode = _.find(this.selectedNodes, { showLabel: str })
      }
      targetNode.checked = false
      this.handleCheck(targetNode)
      this.$emit('remove-tag', v)
    },
    updateSelect (data, needCheckNode = false, setValue = false) {
      let tempSelectedNodes = []
      let tempSelectedLabels = []
      let tempSelectedIds = []
      data.forEach(o => {
        let targetNode
        if (setValue) {
          let findObj = {}
          findObj[this.valueKey] = o
          targetNode = _.find(this.store.nodeList, findObj)
          tempSelectedIds.push(targetNode.id)
        } else {
          targetNode = this.store.nodesMap[o]
          tempSelectedIds.push(o)
        }
        if (targetNode) {
          needCheckNode && targetNode.check(true)
          let label = ''
          if (!this.onlyShowChecked) {
            let level = targetNode.level
            let node = _.cloneDeep(targetNode)
            while (level !== 0) {
              label = node.showLabel + (label ? this.separator : '') + label
              node = node.parent
              level--
            }
          } else {
            label = targetNode.showLabel
          }
          tempSelectedNodes.push(targetNode)
          tempSelectedLabels.push(label)
        }
      })
      this.selectedNodes = tempSelectedNodes
      this.selectedLabels = tempSelectedLabels
      this.selectedIds = tempSelectedIds
    },
    init () {
      this.store = new TreeStore({
        data: this.data,
        isShowIndeterminate: this.isShowIndeterminate,
        selectChildren: this.selectChildren,
        separator: this.separator,
        valueKey: this.valueKey,
        labelKey: this.labelKey,
        childrenKey: this.childrenKey
      })
      this.root = this.store.root
      this.maxLevellist = Array.from({ length: this.store.maxLevel - 1}, (v, i) => {
        this.showData[i + 1] = []
        return {
          id: i + 1,
          rendered: false
        }
      })
      this.updateSelect(this.value, true, true)
    }
  },
  mounted () {
    this.init()
  }
}
</script>

<style lang="scss">
.multi-cascader-base-element {
  .el-tag.el-tag--info.el-tag--small {
    max-width: 100%;
    display: flex;
    align-items: center;
    height: 100%;
    max-height: 60px;
    .el-select__tags-text {
      max-width: 88%;
      height: 100%;
      padding: 2px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: initial;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 3;
      display: -webkit-box;
    }
  }
}
.el-select-dropdown.el-popper.is-multiple.multi-cascader {
  .el-select-dropdown__item.selected::after {
    top: 0;
  }
}
.el-select-dropdown.el-popper.is-multiple.multi-cascader-style{
  min-width: 160px!important;
  .el-select-dropdown__list {
    padding: 0;
    position: relative;
  }
  .el-select-dropdown__item {
    padding: 0;
    height: 204px;
    &.hover {
      background-color: #fff;
    }
  }
  .li-style {
    height: 34px;
    padding: 0px 20px;
    box-sizing: border-box;
    list-style: none;
    width: 160px;
    cursor: pointer;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis; 
    line-height: 34px;
    &:hover{
      background-color: rgba(69,200,220,.1);
    }
    &.selected {
      color: #45c8dc;
    }
    &.active-li {
      background-color: rgba(69,200,220,.1);
      color: #45c8dc;
    }
    .li-label-style{
      text-align: left;
      width: 100%;
      box-sizing: border-box;
      padding-right: 15px;
      position: relative;
      text-overflow: ellipsis;
      white-space: nowrap;
      overflow: hidden;
      .li-label-icon {
        position: absolute;
        right: 0px;
        top: 50%;
        transform: translate(0, -50%);
      }
    }
  }
  .ground {
    width: 100%;
    height: 204px;
    overflow-y: auto;
    overflow-x: hidden;
     ul{
       padding-left: 0px;
       .li-label-style{
         margin-top: 0px;
         margin-bottom: 0px;
       }
     }
  }
  .floor-item {
    width: 160px;
    border-left: 1px solid #eee;
    position: absolute;
    top: 0;
    height: 204px;
    overflow-y: auto;
    overflow-x: hidden;
  }

}
$width: 160px;
@each $i in [1,2,3,4,5,6,7,8,9,10] {
  .multi-cascader-style.floor-width-#{$i} {
    width: $i * $width;
  }
  .multi-cascader-style .floor-position-left-#{$i} {
    left: ($i - 1) * $width;
  }
}
</style>
