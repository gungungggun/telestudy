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
      .buttons(v-show="isShowCanvas")
        button(@click="clear")
          font-awesome-icon(icon="sync")
        button(@click="undo")
          font-awesome-icon(icon="undo")
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
        a: 'ほんよ'
      },
      answers: ['', '', ''],
      canvas: null,
      ctx: null,
      offsetX: 0,
      offsetY: 0,
      lastX: null,
      lastY: null,
      undoDataStack: []
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
          c.beforeDraw()
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
    beforeDraw() {
      if (this.undoDataStack.length >= 10) {
        this.undoDataStack.pop()
      }
      this.undoDataStack.unshift(
        this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height)
      )
    },
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
    undo() {
      if (this.undoDataStack.length <= 0) return
      const imageData = this.undoDataStack.shift()
      this.ctx.putImageData(imageData, 0, 0)
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
      this.activeCanvas(this.activeIndex)
      const canvas = document.createElement('canvas')
      canvas.width = this.canvas.width * 3
      canvas.height = this.canvas.height
      const ctx = canvas.getContext('2d')
      const img = new Image()
      img.src = this.answers[0]
      ctx.drawImage(img, 0, 0)
      img.src = this.answers[1]
      ctx.drawImage(img, this.canvas.width, 0)
      img.src = this.answers[2]
      ctx.drawImage(img, this.canvas.width * 2, 0)
      const mix = canvas.toDataURL('image/png')

      const self = this
      const Tesseract = require('tesseract.js')
      Tesseract.recognize(mix, 'jpn', {
        tessedit_pageseg_mode: Tesseract.PSM.SINGLE_WORD,
        logger: (m) => console.log(m)
      }).then((result) => {
        console.log(result)
        if (result.data.text.replace(/\s+/g, '') === self.question.a) {
          alert('正解')
        } else {
          alert('間違い:' + result.data.text.replace(/\s+/g, ''))
        }
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
        padding 0 10px
        button
          font-size 30px
          margin-right 10px
  .complete
    display flex
    justify-content center
    button
      font-size 30px
      border-radius 20px
      width 180px
      position relative
      display inline-block
      color #FFF
      background #fd9535
      border-bottom solid 2px #d27d00
      border-radius 4px
      box-shadow inset 0 2px 0 rgba(255,255,255,0.2), 0 2px 2px rgba(0, 0, 0, 0.19)
      font-weight bold
      &:active
        border-bottom solid 2px #fd9535
        box-shadow 0 0 2px rgba(0, 0, 0, 0.30)
</style>
