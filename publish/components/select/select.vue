<template>
    <div class="ion-select" :class="[modeClass,{'select-disabled':isDisabled}]">
        <div v-if="!text" class="select-placeholder select-text">{{placeholder}}</div>
        <div v-else class="select-text">{{selectedText || text}}</div>
        <div class="select-icon">
            <div class="select-icon-inner"></div>
        </div>
        <button aria-haspopup="true" @click="onPointerDownHandler($event)"
                :id="id"
                ion-button="item-cover"
                class="item-cover">
        </button>
        <slot></slot>
    </div>
</template>
<style lang="scss">
    @import "select";
    @import "select.ios";
    @import "select.md";
</style>
<script type="text/ecmascript-6">
  /**
   * @name Select
   * @module Component/Select
   * @description
   *
   * 选择组件
   *
   * 如果在Select中使用了`v-model`指令时, Option中的`checked`属性将不起作用, 因为两者的使用逻辑是冲突的!
   * `v-model`是在Select组件中使用数据控制, 而`checked`是在Option中使用checked属性控制.
   *
   * 对外事件:
   * onChange: 选中的值发生变化
   * onCancel: 点击取消
   *
   * */
  import { setElementClass, isTrueProperty, isBlank, isCheckedProperty } from '../../util/util'
  import { ActionSheet } from '../action-sheet'
  import { Alert } from '../alert'
  let id = 0
  export default{
    name: 'Select',
    data(){
      return {
        isDisabled: this.disabled,      // 内部 禁用
        id: `rb-${id++}`,               // id
        itemComponent: null,            // item组件实例
        optionComponents: [],           // Select子组件Option的集合
        texts: [],                      // 回显的数组
        text: null,                     // 回显的string, 已texts为基
        timer: null,                    // setTimeout
        values: [],                     // options中所有选中的value数组
      }
    },
    props: {
      // cancel按钮显示文本
      cancelText: {
        type: String,
        default(){return 'Cancel'}
      },
      // OK按钮显示文本
      okText: {
        type: String,
        default(){return 'OK'}
      },
      disabled: [Boolean],
      // 显示界面类型, 可以是'action-sheet','alert'两个
      interface: {
        type: String,
        default(){return 'alert'}
      },
      // 单选多选,默认为单选
      multiple: {
        type: Boolean,
        default(){return false}
      },
      // 当未选择时显示的值
      placeholder: [String],
      // select组件掉用alert和action-sheet组件的, 这个是针对传入的参数
      // title/subTitle/message/cssClass/enableBackdropDismiss等
      selectOptions: {
        type: Object,
        default(){return {}}
      },
      // 选择组件的文本提示, 代替选择的option选项
      selectedText: [String],
      // 模式
      mode: {
        type: String,
        default(){ return window.VM && window.VM.config.get('mode') || 'ios' }
      },
      value: [Object, String, Array],
    },
    watch: {
      disabled(val){
        this.setDisabled(isTrueProperty(val))
      }
    },
    computed: {
      modeClass () {
        return `select select-${this.mode}`
      }
    },
    methods: {
      /**
       * 设置当前radio的禁用状态
       * */
      setDisabled(isDisabled){
        this.isDisabled = isDisabled
        this.itemComponent && setElementClass(this.itemComponent.$el, 'item-select-disabled', isDisabled)
      },

      /**
       * 点击组件时触发
       * */
      onPointerDownHandler($event){
        $event.preventDefault()
        $event.stopPropagation()
        this.open()
      },

      /**
       * 组件开启
       * */
      open(){
        var selectCssClass
        if (this.isDisabled) {
          return
        }
        console.debug('select, open alert')

        // 避免引用产生问题
        const selectOptions = JSON.parse(JSON.stringify(this.selectOptions))

        // 重新给buttons赋值
        selectOptions.buttons = [{
          text: this.cancelText,
          role: 'cancel',
          handler: () => {
            this.$emit('onCancel', null)
          }
        }]

        // 如果 selectOptions 没有提供title, 则使用Lable的文本
        if (!selectOptions.title && this.itemComponent) {
          selectOptions.title = this.itemComponent.getLabelText()
        }

        if (this.interface === 'action-sheet' && this.optionComponents.length > 6) {
          console.warn('Interface cannot be "action-sheet" with more than 6 options. Using the "alert" interface.')
          this.interface = 'alert'
        }

        if (this.interface === 'action-sheet' && this.multiple) {
          console.warn('Interface cannot be "action-sheet" with a multi-value select. Using the "alert" interface.')
          this.interface = 'alert'
        }

        if (this.interface === 'action-sheet') {
          selectOptions.buttons = selectOptions.buttons.concat(this.optionComponents.map(input => {
            return {
              role: (input.isChecked ? 'selected' : ''),
              text: input.label,
              handler: () => {
                this.onChange(input.optionValue)
                this.$emit('onChange', input.optionValue)
                this.$emit('input', input.optionValue)
                this.$emit('onSelect', input.optionValue)
              }
            }
          }))
          selectCssClass = 'select-action-sheet'

          // 允许用户自定义css样式
          selectCssClass += selectOptions.cssClass ? ' ' + selectOptions.cssClass : ''

          selectOptions.cssClass = selectCssClass

          // 初始化并开启
          ActionSheet.present(selectOptions)

        } else {

          this.interface = 'alert'

          // 从option中获取input参数
          selectOptions.inputs = this.optionComponents.map(input => {
            return {
              type: (this.multiple ? 'checkbox' : 'radio'),
              label: input.label,
              value: input.optionValue,
              checked: input.isChecked,
              disabled: input.disabled,
              handler: (selectedOption) => {
                // 点击选中时触发事件
                if (selectedOption.isChecked) {
                  this.$emit('onSelect', input.optionValue)
                }
              }
            }
          })

          selectCssClass = 'select-alert'

          if (this.multiple) {
            // 使用勾选框组件样式
            selectCssClass += ' multiple-select-alert'
          } else {
            // 使用单选框组件样式
            selectCssClass += ' single-select-alert'
          }

          // 允许用户自定义样式
          selectCssClass += selectOptions.cssClass ? ' ' + selectOptions.cssClass : ''
          selectOptions.cssClass = selectCssClass
          selectOptions.buttons.push({
            text: this.okText,
            handler: (selectedValues) => {
              this.onChange(selectedValues)
              this.$emit('onChange', selectedValues)
              this.$emit('input', selectedValues)
            }
          })

          // 初始化并开启
          Alert.present(selectOptions)
        }
      },

      /**
       * @private
       * 当用户点击选择时
       * */
      onChange(value){
        console.debug('select, onChange value:', value)
        this.values = (Array.isArray(value) ? value : isBlank(value) ? [] : [value])
        this.updOpts()
      },

      /**
       * @private
       * 更新子组件option的状态
       * */
      updOpts(){
        this.texts = []
        if (this.optionComponents) {
          this.optionComponents.forEach(option => {
            // check this option if the option's value is in the values array
            option.isChecked = this.values.some(selectValue => {
              return isCheckedProperty(selectValue, option.optionValue)
            })
            if (option.isChecked) {
              this.texts.push(option.label)
            }
          })
        }
        this.text = this.texts.join(', ')
        console.debug('updOpts')
      },

      /**
       * 由子组件调用, 将自己的this传递给父组件
       * */
      recordOption(optionComponent){
        this.optionComponents.push(optionComponent)
        if (isBlank(this.value)) {
          this.values = this.optionComponents.filter(o => o.isChecked).map(o => o.optionValue)
        }
        this.timer && window.clearTimeout(this.timer)
        // 保证只调用一次
        this.timer = window.setTimeout(() => {
          this.updOpts()
        }, 0)
      }
    },
    created () {},
    mounted () {
      // 找到外部item实例
      if (this.$parent.$options._componentTag.toLowerCase() === 'item') {
        this.itemComponent = this.$parent
        setElementClass(this.itemComponent.$el, 'item-select', true)
      }

      if (!isBlank(this.value)) {
        this.values.push(this.value)
      }
    },
    activated () {},
    components: {}
  }
</script>
