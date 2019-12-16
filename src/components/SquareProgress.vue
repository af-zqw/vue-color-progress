<template>
  <canvas ref="canvasDemo" :width="canvasSize" :height="canvasSize"></canvas>
</template>

<script>
export default {
  data () {
    return {
      stageArr: [0, 12.5, 37.5, 62.5, 87.5, 100],
      aniProgress: 0, // 动画进度
      gradient: null,
      lineX: 0, // 实时的线的x坐标
      lineY: 0 // 实时的线的y坐标
    }
  },
  props: {
    // 进度
    progress: {
      type: Number,
      required: true
    },
    // 正方形边长
    sideLength: {
      type: Number,
      default: 150
    },
    // 正方形线条宽度
    baseWidth: {
      type: Number,
      default: 10
    },
    // 正方形线条颜色
    baseColor: {
      type: String,
      default: '#e5e9f2'
    },
    // 进度条宽度
    lineWidth: {
      type: Number,
      default: 10
    },
    withGradient: { // 进度圆弧是否渐变颜色,需要查看canvas createLinearGradient方法
      type: Boolean,
      default: true
    },
    graColor: { // 渐变颜色数组
      type: Array,
      default: function () {
        return ['#13CDE3', '#3B77E3']
      }
    },
    // 不渐变的进度弧颜色
    lineColor: {
      type: String,
      default: '#42cccc'
    },
    fillet: { // 进度弧是否圆角
      type: Boolean,
      default: true
    },
    isShowPoint: { // 是否开启圆点
      type: Boolean,
      default: true
    },
    pointRadius: { // 圆点半径
      type: Number,
      default: 8
    },
    pointColor: { // 进度点的颜色
      type: String,
      default: '#3B77E3'
    },
    label: { // 中心文字
      type: String
    },
    fontSize: { // 文字大小
      type: Number,
      default: 16
    },
    fontColor: { // 文字颜色
      type: String,
      default: '#42cccc'
    },
    fontWeight: {
      type: Number,
      default: 400
    },
    rate: { // 速度模式
      type: String,
      default: 'normal'
    }
  },
  computed: {
    // 进度阶段
    stage () {
      let step = 0
      if (this.aniProgress >= this.stageArr[1]) {
        step++
      }
      if (this.aniProgress >= this.stageArr[2]) {
        step++
      }
      if (this.aniProgress >= this.stageArr[3]) {
        step++
      }
      if (this.aniProgress >= this.stageArr[4]) {
        step++
      }
      if (this.aniProgress >= this.stageArr[5]) {
        step++
      }
      return step
    },
    // 几个关键点之后多出来的部分
    surplus () {
      return this.aniProgress - this.stageArr[this.stage]
    },
    isBigPoint () { // 判断圆点的大小乘于二跟线条宽度的比较
      if (this.isShowPoint) {
        if (this.pointRadius * 2 > this.baseWidth) {
          return true
        } else {
          return false
        }
      } else {
        return false
      }
    },
    // canvas大小
    canvasSize () {
      if (this.isBigPoint) {
        return this.sideLength + this.pointRadius * 2 + 'px'
      } else {
        return this.sideLength + this.baseWidth + 'px'
      }
    },
    // 正方形各种点坐标
    dotInfo () {
      let origin = this.isBigPoint ? this.pointRadius : this.baseWidth / 2
      let topMiddleX = this.isBigPoint ? (this.sideLength + this.pointRadius * 2) / 2 : (this.sideLength + this.baseWidth) / 2
      let topRightX = this.isShowPoint ? this.sideLength + this.pointRadius : this.sideLength + this.baseWidth / 2
      let downRightY = this.isShowPoint ? this.sideLength + this.pointRadius : this.sideLength + this.baseWidth / 2
      return {
        origin: origin, // 起点x、y坐标
        topMiddleX: topMiddleX, // 上边的中点x坐标，为进度条的起点横坐标
        topRightX: topRightX, // 右上角的点x，第一次拐点
        downRightY: downRightY // 右下角的点y
      }
    },
    lineColorStops () { // 将渐变颜色数组处理成可以给canvas使用的断点颜色数组
      let len = this.graColor.length
      let step = Number((1 / len).toFixed(2))
      let list = []
      let next = 0
      for (let i = 0; i < len; i++) {
        if (i === 0) {
          list.push({percent: 0, color: this.graColor[i]})
          next += step
        } else if (i === len - 1) {
          list.push({percent: 1, color: this.graColor[i]})
        } else {
          list.push({percent: next, color: this.graColor[i]})
          next += step
        }
      }
      return list
    },
    // 速度
    speed () {
      if (this.rate === 'fast') {
        return 2
      } else if (this.rate === 'slow') {
        return 0.5
      } else if (this.rate === 'snail') {
        return 0.1
      } else {
        return 1
      }
    },
    canvasInfo () { // 集合所有props，为了去监听重新绘制canvas
      return {
        progress: this.progress,
        sideLength: this.sideLength,
        baseWidth: this.baseWidth,
        baseColor: this.baseColor,
        lineWidth: this.lineWidth,
        withGradient: this.withGradient,
        graColor: this.graColor,
        lineColor: this.lineColor,
        isShowPoint: this.isBigPoint,
        label: this.label,
        fontSize: this.fontSize,
        fontColor: this.fontColor,
        fontWeight: this.fontWeight,
        rate: this.rate,
        fillet: this.fillet
      }
    }
  },
  methods: {
    initCanvas () {
      var canvas = this.$refs.canvasDemo
      var ctx = canvas.getContext('2d')
      this.amimateDrawArc(canvas, ctx)
      // this.drawSquare(canvas, ctx)
    },
    amimateDrawArc (canvas, ctx) { // 执行动画
      window.requestAnimationFrame(() => {
        ctx.clearRect(0, 0, canvas.clientWidth, canvas.clientHeight)
        this.drawSquare(canvas, ctx)
        this.aniProgress += this.speed
        if (this.aniProgress <= this.progress) {
          this.amimateDrawArc(canvas, ctx)
        }
      })
    },
    drawSquare (canvas, ctx) {
      // 画正方形
      ctx.strokeStyle = this.baseColor
      ctx.lineWidth = this.baseWidth
      ctx.beginPath()
      ctx.rect(this.dotInfo.origin, this.dotInfo.origin, this.sideLength, this.sideLength)
      ctx.stroke()

      // 画进度线
      ctx.lineWidth = this.lineWidth
      if (this.withGradient) {
        // 四个参数分别为渐变开始的x坐标、y坐标，结束的x坐标、y坐标,这里为从上到小
        this.gradient = ctx.createLinearGradient(this.dotInfo.topMiddleX, this.dotInfo.origin, this.dotInfo.topMiddleX, this.dotInfo.downRightY)
        // addColorStop为canvas设置渐变颜色断点的方法
        this.lineColorStops.forEach((item) => {
          this.gradient.addColorStop(item.percent, item.color)
        })
      }
      ctx.strokeStyle = this.withGradient ? this.gradient : this.lineColor
      if (this.fillet) {
        ctx.lineCap = 'round'
      }
      ctx.beginPath()
      ctx.moveTo(this.dotInfo.topMiddleX, this.dotInfo.origin) // 起点
      if (this.stage >= 1) {
        this.lineTo1(ctx)
      }
      if (this.stage >= 2) {
        this.lineTo2(ctx)
      }
      if (this.stage >= 3) {
        this.lineTo3(ctx)
      }
      if (this.stage >= 4) {
        this.lineTo4(ctx)
      }
      if (this.stage >= 5) {
        this.lineTo5(ctx)
      }
      this.drawSurplus(ctx)
      ctx.stroke()
      // 画圆点
      if (this.isShowPoint) {
        ctx.fillStyle = this.pointColor
        ctx.beginPath()
        ctx.arc(this.lineX, this.lineY, this.pointRadius, 0, this.deg2Arc(360))
        ctx.fill()
      }
      if (this.label) {
        ctx.font = `${this.fontWeight} ${this.fontSize}px Arial,"Microsoft YaHei"`
        ctx.fillStyle = this.fontColor
        ctx.textAlign = 'center'
        ctx.textBaseline = 'middle'
        ctx.fillText(this.label, this.dotInfo.topMiddleX, this.dotInfo.topMiddleX)
      }
    },
    lineTo1 (ctx) { // 第一段线，大于等于12.5
      ctx.lineTo(this.dotInfo.topRightX, this.dotInfo.origin) // 第一段线
    },
    lineTo2 (ctx) { // 第二段线，大于等于37.5
      ctx.lineTo(this.dotInfo.topRightX, this.dotInfo.downRightY) // 第二段线
    },
    lineTo3 (ctx) { // 第三段线，大于等于62.5
      ctx.lineTo(this.dotInfo.origin, this.dotInfo.downRightY) // 第三段线
    },
    lineTo4 (ctx) { // 第四段线，大于等于87.5
      ctx.lineTo(this.dotInfo.origin, this.dotInfo.origin) // 第四段线
    },
    lineTo5 (ctx) { // 第五段线，等于100
      ctx.lineTo(this.dotInfo.topMiddleX, this.dotInfo.origin) // 第五段线
    },
    // 处理剩余的线
    drawSurplus (ctx) {
      let x = 0
      let y = 0
      if (this.stage === 0) {
        x = (this.surplus / 12.5) * (this.dotInfo.topRightX - this.dotInfo.topMiddleX) + this.dotInfo.topMiddleX
        y = this.dotInfo.origin
        this.lineX = x
        this.lineY = y
        ctx.lineTo(x, y)
      } else if (this.stage === 1) {
        x = this.dotInfo.topRightX
        y = (this.surplus / 25) * (this.dotInfo.downRightY - this.dotInfo.origin) + this.dotInfo.origin
        this.lineX = x
        this.lineY = y
        ctx.lineTo(x, y)
      } else if (this.stage === 2) {
        x = this.dotInfo.topRightX - (this.surplus / 25) * (this.dotInfo.topRightX - this.dotInfo.origin)
        y = this.dotInfo.downRightY
        this.lineX = x
        this.lineY = y
        ctx.lineTo(x, y)
      } else if (this.stage === 3) {
        x = this.dotInfo.origin
        y = this.dotInfo.downRightY - (this.surplus / 25) * (this.dotInfo.downRightY - this.dotInfo.origin)
        this.lineX = x
        this.lineY = y
        ctx.lineTo(x, y)
      } else if (this.stage === 4) {
        x = (this.surplus / 12.5) * (this.dotInfo.topMiddleX - this.dotInfo.origin) + this.dotInfo.origin
        y = this.dotInfo.origin
        this.lineX = x
        this.lineY = y
        ctx.lineTo(x, y)
      }
    },
    // 将角度转成弧度
    deg2Arc (deg) {
      return deg / 180 * Math.PI
    }
  },
  watch: {
    canvasInfo: {
      handler: function (val) {
        this.aniProgress = 0
        this.initCanvas()
      },
      deep: true
    }
  },
  mounted () {
    this.$nextTick(() => {
      this.initCanvas()
    })
  }
}
</script>
