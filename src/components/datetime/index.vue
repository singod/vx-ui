<template>
  <x-input
    :class="{'is-focus': isFocus}"
    :type="nativeType"
    :placeholder="placeholder"
    readonly="readonly"
    :value="value"
    :disabled="disabled"
    :autofocus="autofocus"
    :maxlength="maxlength"
    :name="name"
    :required="required"
    :format="format"
    :clear="clear"
    :arrow="true"
    @focus="handleFocus"
    @blur="handleBlur"
    @keyup="handleKeyup"
    @keydown="handleKeydown"
    @change="handleChange"
    @input="handleInput"
    @click.native="handleClick"
    >
    <slot name="prepend" slot="prepend"></slot>
    <slot name="append" slot="append"></slot>
  </x-input>
</template>

<script>
import { input } from 'utils/mixins'
import Vue from 'vue'
import XInput from '../input'
import DatetimePicker from '../datetime-picker'
export default {
  componentName: 'Datetime',
  mixins: [input],
  components: {
    XInput
  },
  props: {
    ...input.props,
    format: {
      type: String,
      default: 'yyyy-MM-dd'
    },
    clear: {
      type: Boolean,
      default: false
    },
    getPopupMounted: {
      type: Function
    }
  },
  data () {
    return {
      isFocus: false
    }
  },
  methods: {
    handleClick (e) {
      let datetime = this
      let node = document.createElement('div')
      if (this.getPopupMounted) {
        this.getPopupMounted(e).appendChild(node)
      } else {
        document.body.appendChild(node)
      }
      /* eslint-disable no-new */
      this.$$datetimePicker = new Vue({
        el: node,
        render (createElement) {
          return createElement(DatetimePicker, {
            props: {
              open: this.open,
              value: this.value,
              format: this.format
            },
            on: {
              'change': this.handleChange,
              'close': this.handleClose,
              'close-after': this.handleCloseAfter
            }
          })
        },
        components: { DatetimePicker },
        data () {
          let now = new Date()
          return {
            open: false,
            value: datetime.value || now.format(datetime.format),
            format: datetime.format
          }
        },
        mounted () {
          requestAnimationFrame(() => {
            this.open = true
            datetime.isFocus = true
          })
        },
        destroyed () {
          this.$el.parentNode && this.$el.parentNode.removeChild(this.$el)
        },
        methods: {
          handleChange (value) {
            this.open = false
            datetime.$emit('input', value).$emit('change', value)
            datetime.eDispatch('ElFormItem', 'el.form.blur', [value])
            datetime.eDispatch('ElFormItem', 'el.form.change', [value])
          },
          handleClose () {
            this.open = false
            datetime.isFocus = false
          },
          handleCloseAfter () {
            this.$destroy()
          }
        }
      })
    }
  },
  destroyed () {
    this.$$datetimePicker && this.$$datetimePicker.$destroy()
  }
}
</script>

