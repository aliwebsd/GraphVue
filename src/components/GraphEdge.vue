<template>
  <canvas
    class="edge-line"
    ref="edgeLine"
    width="1161px"
    height="368px"
  ></canvas>
</template>

<script>
import Vue from 'vue'
export default Vue.extend({
  data: () => ({
    top: '',
    left: '',
    context: null
  }),
  props: {
    source: String,
    target: String,
    color: {
      type: String,
      default: '#7e7e7e'
    }
  },
  methods: {
    drawArrow (p1, p2, size, ctx) {
      var angle = Math.atan2((p2.y - p1.y) , (p2.x - p1.x))
      var hyp = Math.sqrt((p2.x - p1.x) * (p2.x - p1.x) + (p2.y - p1.y) * (p2.y - p1.y))
      ctx.save()
      ctx.translate(p1.x, p1.y)
      ctx.rotate(angle)

      // line
      ctx.strokeStyle = this.color
      ctx.beginPath()
      ctx.moveTo(0, 0)
      ctx.lineTo(hyp - size, 0)
      ctx.stroke()

      // triangle
      ctx.fillStyle = this.color
      ctx.beginPath()
      ctx.lineTo(hyp - size, size)
      ctx.lineTo(hyp, 0)
      ctx.lineTo(hyp - size, -size)
      ctx.fill()

      ctx.restore()
    },
    init () {
      const source = this.source ? document.querySelector(`#node-${this.source}`) : null
      const target = this.target ? document.querySelector(`#node-${this.target}`) : null
      const canvas = this.$refs.edgeLine
      if (!this.context) {
        this.context = canvas.getContext('2d')
      }
      if (source && target && canvas) {
        this.context.clearRect(0, 0, canvas.width, canvas.height);
        this.drawArrow({x: source.offsetLeft, y: source.offsetTop}, {x: target.offsetLeft, y: target.offsetTop-22}, 10, this.context)
      }
    }
  },
  mounted() {
    this.$watch(vm => [vm.source, vm.target, vm.color], () => {
      setTimeout(() => {
        this.init()
      })
    }, {
      immediate: true
    })
  }
})
</script>

<style lang="scss">
.edge-line {
  position: absolute;
  inset: 0;
}
</style>
