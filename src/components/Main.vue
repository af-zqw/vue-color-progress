<template>
  <div>
    <circle-progress :progress="progress"
                     :circleRadius="circleRadius"
                     :baseWidth="baseWidth"
                     :baseColor="baseColor"
                     :lineWidth="lineWidth"
                     :fillet="fillet"
                     :withGradient="withGradient"
                     :graColor="graColor"
                     :lineColor="lineColor"
                     :isShowPoint="isShowPoint"
                     :pointRadius="pointRadius"
                     :pointColor="pointColor"
                     :label="label"
                     :fontSize="fontSize"
                     :fontColor="fontColor"
                     :fontWeight="fontWeight"
                     :rate="rate"
                     v-if="type=='circle'"
                      >
    </circle-progress>
    <square-progress :progress="progress"
                     :sideLength="sideLength"
                     :baseWidth="baseWidth"
                     :baseColor="baseColor"
                     :lineWidth="lineWidth"
                     :fillet="fillet"
                     :withGradient="withGradient"
                     :graColor="graColor"
                     :lineColor="lineColor"
                     :isShowPoint="isShowPoint"
                     :pointRadius="pointRadius"
                     :pointColor="pointColor"
                     :label="label"
                     :fontSize="fontSize"
                     :fontColor="fontColor"
                     :fontWeight="fontWeight"
                     :rate="rate"
                     v-if="type=='square'"
                      >
    </square-progress>
  </div>
</template>

<script>
import CircleProgress from './CircleProgress.vue'
import SquareProgress from './SquareProgress.vue'
export default {
  components: {
    CircleProgress,
    SquareProgress
  },
  props: {
    type: {
      type: String,
      default: 'circle'
    },
    progress: {
      type: Number,
      required: true
    },
    circleRadius: { // 圆环半径
      type: Number,
      default: 80
    },
    // 正方形边长
    sideLength: {
      type: Number,
      default: 150
    },
    // 底图线条宽度
    baseWidth: {
      type: Number,
      default: 10
    },
    // 底图线条颜色
    baseColor: {
      type: String,
      default: '#e5e9f2'
    },
    // 进度条宽度
    lineWidth: {
      type: Number,
      default: 10
    },
    // 进度条是否圆角
    fillet: {
      type: Boolean,
      default: true
    },
    // 进度圆弧是否渐变颜色,需要查看canvas createLinearGradient方法
    withGradient: {
      type: Boolean,
      default: true
    },
    // 渐变颜色数组
    graColor: {
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
    // 是否开启圆点
    isShowPoint: {
      type: Boolean,
      default: false
    },
    // 圆点半径
    pointRadius: {
      type: Number,
      default: 8
    },
    // 进度点的颜色
    pointColor: {
      type: String,
      default: '#3B77E3'
    },
    // 中心文字
    label: {
      type: String
    },
    // 文字大小
    fontSize: {
      type: Number,
      default: 16
    },
    // 文字颜色
    fontColor: { 
      type: String,
      default: '#42cccc'
    },
    // 文字粗细
    fontWeight: {
      type: Number,
      default: 400
    },
    // 速度模式
    rate: {
      type: String,
      default: 'normal'
    }
  }
}
</script>