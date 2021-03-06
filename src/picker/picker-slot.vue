<template>
  <div class="picker-slot" :class="{'picker-slot-divider': divider}" :style="{'width': width}">
    <div v-if="!divider" class="picker-slot-wrapper" :style="{'height': contentHeight + 'px'}" v-el:wrapper>
      <div class="picker-item" :style="{'text-align': textAlign}" :class="{'selected': item === value}"
      v-for="item in values">{{item.text || item}}</div>
    </div>
    <div v-if="divider">{{ content }}</div>
  </div>
</template>

<script>
import Vue from 'vue'
import Drag from '../utils/drag'
import translateUtil from '../utils/translate'
const ITEM_HEIGHT = 36
export default {
  props: {
    divider: {
      type: Boolean,
      default: false
    },
    content: {
      type: String,
      default: ''
    },
    values: {
      type: Array,
      default () {
        return []
      }
    },
    value: {},
    textAlign: {
      type: String,
      default: ''
    },
    width: {
      type: String,
      default: ''
    },
    visibleItemCount: {
      type: Number,
      default: 5
    }
  },
  data () {
    return {
      dragging: false
    }
  },
  computed: {
    contentHeight () {
      return ITEM_HEIGHT * this.visibleItemCount
    },
    valueIndex () {
      return this.values.indexOf(this.value)
    },
    dragRange () {
      let values = this.values
      let visibleItemCount = this.visibleItemCount
      return [ -ITEM_HEIGHT * (values.length - Math.ceil(visibleItemCount / 2)), ITEM_HEIGHT * Math.floor(visibleItemCount / 2) ]
    }
  },
  ready: function () {
    if (!this.divider) {
      this.initEvents()
      this.doOnValueChange()
    }
  },
  methods: {
    value2Translate (value) {
      let values = this.values
      let valueIndex = values.indexOf(value)
      let offset = Math.floor(this.visibleItemCount / 2)
      if (valueIndex !== -1) {
        return (valueIndex - offset) * -ITEM_HEIGHT
      }
    },
    translate2Value (translate) {
      translate = Math.round(translate / ITEM_HEIGHT) * ITEM_HEIGHT
      let index = -(translate - Math.floor(this.visibleItemCount / 2) * ITEM_HEIGHT) / ITEM_HEIGHT
      return this.values[index]
    },
    doOnValueChange () {
      let value = this.value
      let wrapper = this.$els.wrapper
      translateUtil.translateElement(wrapper, null, this.value2Translate(value))
    },
    doOnValuesChange () {
      let el = this.$el
      let items = el.querySelectorAll('.picker-item')
      Array.prototype.forEach.call(items, (item, index) => {
        translateUtil.translateElement(item, null, ITEM_HEIGHT * index)
      })
    },
    initEvents () {
      let el = this.$els.wrapper
      let drag = new Drag(el)
      let startTop = 0
      let velocityTranslate, prevTranslate
      drag.start(() => {
        startTop = translateUtil.getElementTranslate(el).top
      }).drag((endPos, event) => {
        event.preventDefault()
        event.stopPropagation()
        let translate = startTop + endPos.y
        translateUtil.translateElement(el, 0, translate)
        velocityTranslate = translate - prevTranslate || translate
        prevTranslate = translate
      }).end((endPos) => {
        const momentumRatio = 7
        let currentTranslate = translateUtil.getElementTranslate(el).top
        let momentumTranslate
        if (endPos.time < 300) {
          momentumTranslate = currentTranslate + velocityTranslate * momentumRatio
        }
        let dragRange = this.dragRange
        Vue.nextTick(() => {
          let translate
          if (momentumTranslate) {
            translate = Math.round(momentumTranslate / ITEM_HEIGHT) * ITEM_HEIGHT
          } else {
            translate = Math.round(currentTranslate / ITEM_HEIGHT) * ITEM_HEIGHT
          }
          translate = Math.max(Math.min(translate, dragRange[1]), dragRange[0])
          translateUtil.translateElement(el, null, translate)

          this.value = this.translate2Value(translate)
        })
      })
    }
  },
  watch: {
    values (newVal) {
      if (this.valueIndex === -1) {
        this.value = (newVal || [])[0]
      }
    },
    value () {
      this.doOnValueChange()
      this.$emit('slot-select', this.value)
    }
  },
  components: {}
}
</script>

<style lang="less">
@import "../utils/_vars.less";
@import "../utils/_mixins.less";
.picker-slot{
  .flex-shrink(1);
  font-size: 18px;
  overflow: hidden;
  position: relative;
  max-height: 100%;
  text-align: center;
  &.picker-slot-divider{
    color: #000;
    display: flex;
    align-items: center;
    line-height: 36px;
  }
}
.picker-slot-wrapper{
  transition-duration: .3s;
  transition-timing-function: ease-out;
  backface-visibility: hidden;
}
.picker-item{
  line-height: 36px;
  padding: 0 10px;
  white-space: nowrap;
  position: relative;
  overflow: hidden;
  text-overflow: ellipsis;
  color: @body_color;
  left: 0;
  top: 0;
  width: 100%;
  box-sizing: border-box;
  transition-duration: .3s;
  backface-visibility: hidden;
  &.selected {
    color: @color;
    transform: translate3d(0, 0, 0) rotateX(0);
  }
}
</style>
