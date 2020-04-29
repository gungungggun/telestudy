<template lang="pug">
.page-top
  canvas(v-draw)
  button(@click="clear") clear
  button(@click="answer") answer
</template>

<script>
export default {
  data() {
    return {
      canvas: null,
      ctx: null,
      offsetY: 0,
      lastX: null,
      lastY: null
    }
  },
  directives: {
    draw: {
      bind(el, binding, vnode) {
        const c = vnode.context
        el.width = window.innerWidth
        el.height = window.innerHeight - c.offsetY - 50
        c.canvas = el
        c.ctx = el.getContext('2d')
        el.addEventListener('touchstart', (e) => {
          const x = e.changedTouches[0].pageX
          const y = e.changedTouches[0].pageY - c.offsetY
          c.draw(x, y)
        })
        el.addEventListener('touchmove', (e) => {
          const x = e.changedTouches[0].pageX
          const y = e.changedTouches[0].pageY - c.offsetY
          c.draw(x, y)
        })
        el.addEventListener('touchend', (e) => {
          c.lastX = null
          c.lastY = null
        })
      }
    }
  },
  methods: {
    draw(x, y) {
      this.ctx.beginPath()
      if (this.lastX == null) {
        this.ctx.moveTo(x, y)
      } else {
        this.ctx.moveTo(this.lastX, this.lastY)
      }
      this.ctx.lineTo(x, y)
      this.ctx.lineCap = 'round'
      this.ctx.lineWidth = 5
      this.ctx.strokeStyle = '#000'
      this.ctx.stroke()
      this.lastX = x
      this.lastY = y
    },
    clear() {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height)
    },
    answer() {
      const Tesseract = require('tesseract.js')
      const base64 = this.canvas.toDataURL('image/png')
      Tesseract.recognize(base64, 'jpn', {
        tessedit_pageseg_mode: Tesseract.PSM.SINGLE_WORD,
        logger: (m) => console.log(m)
      }).then((result) => {
        console.log(result)
      })
    }
  }
}
</script>

<style lang="stylus">
.page-top
  margin 0 auto
</style>
