<template lang="pug">
.page-top
  canvas(v-draw)
  button(@click="clear") clear
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
    update() {
      const base64 = this.canvas.toDataURL('image/png')
      console.log(base64)
    }
  }
}
</script>

<style lang="stylus">
.page-top
  margin 0 auto
</style>
