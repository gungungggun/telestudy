<template lang="pug">
.page-top
  .question
    .q
      p {{ question.q }}
    .kana
      .answer(v-for="(qb, index1) in question.b")
        .rec(v-for="(b, index2) of qb" @click="activeCanvas(index1 + index2)" :class="{ active: activeIndex==index1 + index2 }")
          img(v-if="answers[index1 + index2] != ''" :src="answers[index1 + index2]")
    .draw(ref="draw")
      canvas(v-draw v-show="isShowCanvas")
      .buttons
        button(@click="clear") ぜんぶかきなおす
  .complete
    button(@click="answer") できた
</template>

<script>
export default {
  data() {
    return {
      isShowCanvas: false,
      activeIndex: null,
      question: {
        q: '本を読む',
        b: [2, 0, 1, 0],
        a: ['ほん', 'よ']
      },
      answers: ['', '', ''],
      canvas: null,
      ctx: null,
      offsetX: 0,
      offsetY: 0,
      lastX: null,
      lastY: null
    }
  },
  directives: {
    draw: {
      bind(el, binding, vnode) {
        const c = vnode.context
        setTimeout(() => {
          el.width = c.$refs.draw.clientWidth
          el.height = c.$refs.draw.clientHeight
          const rect = c.$refs.draw.getBoundingClientRect()
          c.offsetX = rect.left
          c.offsetY = rect.top
        }, 100)
        c.canvas = el
        c.ctx = el.getContext('2d')
        el.addEventListener('touchstart', (e) => {
          const x = e.changedTouches[0].pageX - c.offsetX
          const y = e.changedTouches[0].pageY - c.offsetY
          c.draw(x, y)
        })
        el.addEventListener('touchmove', (e) => {
          const x = e.changedTouches[0].pageX - c.offsetX
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
    load(index) {
      const img = new Image()
      img.src = this.answers[index]
      this.ctx.drawImage(img, 0, 0)
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
    },
    activeCanvas(index) {
      this.isShowCanvas = true
      const base64 = this.canvas.toDataURL('image/png')
      if (this.activeIndex !== null) {
        this.answers[this.activeIndex] = base64
      }
      this.activeIndex = index
      this.clear()
      this.load(index)
    }
  }
}
</script>

<style lang="stylus">
.page-top
  margin 0 auto
  .question
    display flex
    padding 15px 10px
    .q
      width 80px
      p
        font-size 80px
        vertical-write()
    .kana
      width 50px
      height auto
      display flex
      flex-direction column
      justify-content space-around
      .answer
        width 30px
        height 80px
        display flex
        flex-direction column
        justify-content center
        .rec
          width 30px
          height 30px
          border 1px solid #ccc
          img
            width 100%
            height 100%
          &.active
            border-color #f00
    .draw
      position relative
      margin-top 10px
      width calc(100vw - 80px - 50px - 20px)
      height calc(100vw - 80px - 50px - 20px + 50px)
      border 1px solid #ccc
      .buttons
        position absolute
        bottom 0
</style>
