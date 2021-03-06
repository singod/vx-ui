<template>
  <x-input
    :type="nativeType"
    :placeholder="placeholder"
    readonly="readonly"
    :value="myValue"
    :clear="clear"
    :disabled="disabled"
    :autofocus="autofocus"
    :maxlength="maxlength"
    :name="name"
    :required="required"
    :format="format"
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
import DaterangePicker from '../daterange-picker'
import XInput from '../input'
export default {
  componentName: 'Daterange',
  components: {
    XInput
  },
  mixins: [input],
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
    value: {
      type: Array,
      default () {
        return []
      }
    },
    separator: {
      type: String,
      default: ' ~ '
    },
    getPopupMounted: {
      type: Function
    }
  },
  computed: {
    myValue () {
      if (this.value && this.value[0] && this.value[1]) {
        return this.value[0].format() + this.separator + this.value[1].format()
      }
      return ''
    }
  },
  methods: {
    handleClick (e) {
      let daterange = this
      let node = document.createElement('div')
      if (this.getPopupMounted) {
        this.getPopupMounted(e).appendChild(node)
      } else {
        document.body.appendChild(node)
      }
      let value = []
      if (this.value && this.value[0] && this.value[1]) {
        value = [new Date(this.value[0].getFullYear(), this.value[0].getMonth(), this.value[0].getDate()), new Date(this.value[1].getFullYear(), this.value[1].getMonth(), this.value[1].getDate())]
      }
      /* eslint-disable no-new */
      this.$$daterangePicker = new Vue({
        el: node,
        render (createElement) {
          return createElement(DaterangePicker, {
            props: {
              value: this.value,
              open: this.open
            },
            on: {
              'change': this.handleChange,
              'close': this.handleClose,
              'close-after': this.handleCloseAfter
            }
          })
        },
        components: { DaterangePicker },
        data: {
          open: false,
          value: value
        },
        mounted () {
          requestAnimationFrame(() => {
            this.open = true
          })
        },
        destroyed () {
          this.$el.parentNode && this.$el.parentNode.removeChild(this.$el)
        },
        methods: {
          handleChange (value) {
            daterange.$emit('input', value).$emit('change', value)
            daterange.eDispatch('ElFormItem', 'el.form.blur', [value])
            daterange.eDispatch('ElFormItem', 'el.form.change', [value])
          },
          handleClose () {
            this.open = false
          },
          handleCloseAfter () {
            this.$nextTick(() => {
              this.$destroy()
            })
          }
        }
      })
    }
  },
  destroyed () {
    this.$$daterangePicker && this.$$daterangePicker.$destroy()
  }
}
</script>

