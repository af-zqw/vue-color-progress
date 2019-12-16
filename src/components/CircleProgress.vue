<template>
  <canvas class="canvas" ref="canvasDemo" :width="canvasSize" :height="canvasSize"></canvas>
</template>

<script>
/**
 * canvasSize: canvas宽度/高度
 * outerRadius: 外围半径
 * pointRadius: 圆点半径(圆上面的进度点)
 * circleRadius: 圆环半径(大圆)
 * canvasSize = 2 * outerRadius = 2 * (pointRadius + circleRadius)
 */

export default {
  data () {
    return {
      gradient: null,
      currentNum: 0 // 当前画图的次数
    }
  },
  props: {
    progress: { // 进度，必填，转化为角度
      type: Number,
      required: true
    },
    circleRadius: { // 圆环半径
      type: Number,
      default: 80
    },
    pointRadius: { // 圆点半径
      type: Number,
      default: 8
    },
    isShowPoint: { // 是否开启圆点
      type: Boolean,
      default: false
    },
    baseWidth: { // 圆环线宽
      type: Number,
      default: 10
    },
    baseColor: { // 圆环颜色
      type: String,
      default: '#e5e9f2'
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
    lineColor: { // 不渐变的进度弧颜色
      type: String,
      default: '#42cccc'
    },
    lineWidth: { // 进度弧宽度
      type: Number,
      default: 10
    },
    fillet: { // 进度弧是否圆角
      type: Boolean,
      default: true
    },
    pointColor: { // 进度点的颜色
      type: String,
      default: '#3B77E3'
    },
    rate: { // 速度模式
      type: String,
      default: 'normal'
    }
    // speed: { // 速度
    //   type: Number,
    //   default: 5
    // }
  },
  computed: {
    // 外围半径等于圆环半径加上圆点半径或者进度弧的宽度
    outerRadius () {
      if (this.isShowPoint) { // 如果开启圆点
        if (this.lineWidth / 2 > this.pointRadius) { // 判断进度弧的宽度和圆点的宽度谁宽
          return this.circleRadius + this.lineWidth / 2
        } else {
          return this.circleRadius + this.pointRadius
        }
      } else {
        return this.circleRadius + this.lineWidth / 2 // 要加上进度弧的宽度
      }
    },
    // canvas宽高
    canvasSize () {
      return this.outerRadius * 2 + 2 + 'px'
    },
    // 进度弧起止角度
    angleRange () {
      let result = parseInt(this.progress / 100 * 360)
      if (result < 90) {
        return [270, result + 270]
      } else {
        return [270, result - 90]
      }
    },
    // 画图的次数
    total () {
      if (this.angleRange[1] > 270) {
        return Math.ceil((this.angleRange[1] - 270) / this.speed)
      } else {
        return Math.ceil((this.angleRange[1] + 90) / this.speed)
      }
    },
    // 速度
    speed () {
      if (this.rate === 'fast') {
        return 10
      } else if (this.rate === 'slow') {
        return 2
      } else if (this.rate === 'snail') {
        return 0.5
      } else {
        return 5
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
    canvasInfo () { // 集合所有props，为了去监听重新绘制canvas
      return {
        progress: this.progress,
        circleRadius: this.circleRadius,
        pointRadius: this.pointRadius,
        isShowPoint: this.isShowPoint,
        baseWidth: this.baseWidth,
        baseColor: this.baseColor,
        label: this.label,
        fontSize: this.fontSize,
        fontColor: this.fontColor,
        withGradient: this.withGradient,
        lineColor: this.lineColor,
        lineWidth: this.lineWidth,
        pointColor: this.pointColor,
        rate: this.rate,
        fontWeight: this.fontWeight,
        fillet: this.fillet
      }
    }
  },
  methods: {
    initCanvas () { // 初始化canvas
      var canvas = this.$refs.canvasDemo
      var ctx = canvas.getContext('2d')
      // this.drawCirle(canvas, ctx)
      var endDeg = this.getInitEndDeg(this.angleRange[1])
      this.currentNum = 0
      this.amimateDrawArc(canvas, ctx, this.angleRange[0], endDeg) // 执行画图的方法
    },
    getInitEndDeg (deg) { // 取得第一次结束的角度
      let endDeg = deg
      if (deg > 270 && deg < 360) {
        if ((deg - 270) > this.speed) {
          endDeg = 270 + this.speed
        } else {
          endDeg = deg
        }
      } else {
        if (270 + this.speed >= 360) {
          if ((270 + this.speed - 360) >= deg) {
            endDeg = deg
          } else {
            endDeg = 270 + this.speed - 360
          }
        } else {
          endDeg = 270 + this.speed
        }
      }
      return endDeg
    },
    getNextDeg (endDeg) { // 取得每次画圆结束的角度
      if (this.angleRange[1] > 270) {
        return (endDeg + this.speed) >= this.angleRange[1] ? this.angleRange[1] : (endDeg + this.speed)
      }
      if (this.angleRange[1] <= 270) {
        if ((endDeg + this.speed) > 270 && (endDeg + this.speed) < 360) {
          return (endDeg + this.speed)
        } else if ((endDeg + this.speed) >= 360) {
          return (endDeg + this.speed) - 360
        } else if ((endDeg + this.speed) < 270) {
          return (endDeg + this.speed) >= this.angleRange[1] ? this.angleRange[1] : (endDeg + this.speed)
        } else if ((endDeg + this.speed) === 270) {
          return -90
        }
      }
    },
    amimateDrawArc (canvas, ctx, startDeg, endDeg) {
      window.requestAnimationFrame(() => {
        // 清空画布
        ctx.clearRect(0, 0, canvas.clientWidth, canvas.clientHeight)
        this.drawCirle(canvas, ctx, startDeg, endDeg)

        if (this.currentNum < this.total) {
          let nextDeg = this.getNextDeg(endDeg)
          this.amimateDrawArc(canvas, ctx, startDeg, nextDeg)
        }
      })
    },
    /**
     * canvas提供的画圆弧的方法是ctx.arc()，需要提供圆心坐标，半径radius，起止弧度，是否逆时针等参数。
     * ctx.arc(x, y, radius, startAngle, endAngle, anticlockwise);
     * 对于角度而言，0°是x轴正向，默认是顺时针方向旋转。
     * 圆环的圆心就是canvas的中心，所以x, y 取outerRadius的值就可以了。
     */
    drawCirle (canvas, ctx, startDeg, endAngle) {
      this.currentNum++
      // 画圆
      ctx.strokeStyle = this.baseColor // 画笔颜色
      ctx.lineWidth = this.baseWidth // 画笔粗细
      ctx.beginPath()
      ctx.arc(this.outerRadius, this.outerRadius, this.circleRadius, 0, this.deg2Arc(360))
      ctx.stroke()
      // 注意，零度是从圆心往x轴正向开始的.
      /**
       * 调用fillText绘制文字，利用canvas.clientWidth / 2和canvas.clientWidth / 2取得中点坐标
       * 结合控制文字对齐的两个属性textAlign和textBaseline，我们可以将文字绘制在画布中央。
       * 文字的值由label属性接收，字体大小由fontSize属性接收，颜色则取的fontColor。
       */
      if (this.label) {
        ctx.font = `${this.fontWeight} ${this.fontSize}px Arial,"Microsoft YaHei"`
        ctx.fillStyle = this.fontColor
        ctx.textAlign = 'center'
        ctx.textBaseline = 'middle'
        ctx.fillText(this.label, canvas.clientWidth / 2, canvas.clientWidth / 2)
      }

      // 画进度弧
      if (this.withGradient) {
        // 四个参数分别为渐变开始的x坐标、y坐标，结束的x坐标、y坐标,这里为从上到小
        this.gradient = ctx.createLinearGradient(this.circleRadius, 0, this.circleRadius, this.circleRadius * 2)
        // addColorStop为canvas设置渐变颜色断点的方法
        this.lineColorStops.forEach((item) => {
          this.gradient.addColorStop(item.percent, item.color)
        })
      }
      ctx.strokeStyle = this.withGradient ? this.gradient : this.lineColor
      ctx.lineWidth = this.lineWidth
      ctx.beginPath()
      if (this.fillet) {
        ctx.lineCap = 'round'
      }
      ctx.arc(this.outerRadius, this.outerRadius, this.circleRadius, this.deg2Arc(startDeg), this.deg2Arc(endAngle))
      ctx.stroke()

      // 画圆点
      if (this.isShowPoint) {
        ctx.fillStyle = this.pointColor
        ctx.beginPath()
        const pointPosition = this.getPositionsByDeg(endAngle)
        ctx.arc(pointPosition.x + this.pointRadius, pointPosition.y + this.pointRadius, this.pointRadius, 0, this.deg2Arc(360))
        ctx.fill()
      }
    },
    // 将角度转成弧度
    deg2Arc (deg) {
      return deg / 180 * Math.PI
    },
    // 求得圆点的坐标
    getPositionsByDeg (deg) {
      let x = 0
      let y = 0
      if (deg >= 0 && deg <= 90) {
        // 0~90度
        x = this.circleRadius * (1 + Math.cos(this.deg2Arc(deg)))
        y = this.circleRadius * (1 + Math.sin(this.deg2Arc(deg)))
      } else if (deg > 90 && deg <= 180) {
        // 90~180度
        x = this.circleRadius * (1 - Math.cos(this.deg2Arc(180 - deg)))
        y = this.circleRadius * (1 + Math.sin(this.deg2Arc(180 - deg)))
      } else if (deg > 180 && deg <= 270) {
        // 180~270度
        x = this.circleRadius * (1 - Math.sin(this.deg2Arc(270 - deg)))
        y = this.circleRadius * (1 - Math.cos(this.deg2Arc(270 - deg)))
      } else {
        // 270~360度
        x = this.circleRadius * (1 + Math.cos(this.deg2Arc(360 - deg)))
        y = this.circleRadius * (1 - Math.sin(this.deg2Arc(360 - deg)))
      }
      return { x, y }
    }
  },
  mounted () {
    // 在$nextTick初始化画布，不然dom还未渲染好
    this.$nextTick(() => {
      this.initCanvas()
    })
  },
  watch: {
    canvasInfo: {
      handler: function (val) {
        this.initCanvas()
      },
      deep: true
    }
  }
}
</script>
