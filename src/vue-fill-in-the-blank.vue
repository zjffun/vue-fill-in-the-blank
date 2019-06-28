<template>
  <span contenteditable="true" class="blank-input" ref="bi"></span>
</template>

<script>
export default {
  name: 'VueFillInTheBlank', // vue component name
  props: {
    useMinLength: {
      type: Boolean,
      default: true
    },
    blankChars: {
      type: Array,
      default: () => [8195] //&#x2003;
    },
    minLength: {
      type: Number,
      required: false
    },
    maxLength: {
      type: Number,
      required: false
    }
  },

  data: () => ({
    selfMinLength: 0,
    selfMaxLength: 0
  }),

  computed: {
    isIE: () => /Trident/.test(navigator.userAgent)
  },

  mounted() {
    let _this = this
    let bi = this.$refs.bi
    let composing = false

    // content
    bi.innerHTML = `${this.$slots.default[0].text}${this.isIE ? '&nbsp;' : ''}`

    // IE not support input event of contenteditable element
    if (this.isIE) {
      // bi.addEventListener("keydown", e => {
      // })
      return
    }

    // min&max length
    this.selfMinLength =
      typeof this.minLength !== 'undefined'
        ? this.minLength
        : bi.innerText.length
    this.selfMaxLength =
      typeof this.maxLength !== 'undefined'
        ? this.maxLength
        : bi.innerText.length

    // 处理组合的输入（语音、拼音）
    bi.addEventListener('compositionstart', _ => {
      composing = true
    })

    bi.addEventListener('compositionend', e => {
      _this.inputChars()
      composing = false
    })

    // input event
    bi.addEventListener('input', e => {
      if (this.useMinLength) {
        switch (e.inputType) {
          case 'deleteContentBackward':
            _this.inputChars()
            break
          default:
            !composing && _this.inputChars()
            break
        }
      }
      this.$emit('input', bi.innerText)
    })
  },

  methods: {
    inputChars() {
      let selection = window.getSelection()
      let offset = selection.anchorOffset
      let node = selection.anchorNode
      let remove = node.nodeValue.length - this.selfMaxLength
      let fill = this.selfMinLength - node.nodeValue.length
      if (remove > 0) {
        let blankNum = getLastBlankNum(node.nodeValue, this.blankChars)
        if (blankNum > 0) {
          node.nodeValue = node.nodeValue.substring(
            0,
            node.nodeValue.length - Math.min(remove, blankNum)
          )
          selection.collapse(node, offset)
        }
        return
      }
      if (fill > 0) {
        node.nodeValue =
          node.nodeValue + String.fromCharCode(this.blankChars[0]).repeat(fill)
        selection.collapse(node, offset)
      }
    }
  }
}

function getLastBlankNum(str, blankChars) {
  // trim???
  let reg = new RegExp(`[${blankChars.map(String.fromCharCode).join('')}]*$`)
  let result = reg.exec(str)
  return result && result[0] ? result[0].length : 0
}
</script>

<style lang="stylus" scoped>
.blank-input {
  text-decoration: underline;
}
</style>
