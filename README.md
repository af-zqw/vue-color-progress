# vue-color-progress

> A Vue.js project

## Build Setup

``` bash
# 安装
npm install vue-color-progress

# 引入
import ColorProgress from 'vue-color-progress'

# 使用(圆形)
<color-progress :progress="50"></color-progress>

# 使用(正方形)
<color-progress :progress="50" type="square"></color-progress>

```
## 配置项

名称 | 解释 | 是否必填 | 类型 | 默认值 | 可选值 |
---- | ---- | ---- | ---- | ---- | ---- |
progress | 进度 | 是 | Number |  |  |
type | 进度条类型 | 否 | String | 'circle' | 'circle'、'square' |
circleRadius | 圆环半径(选择circle类型时候) | 否 | Number | 60 | |
sideLength | 正方形边长(选择square类型时候) | 否 | Number | 150 | |
baseWidth | 底图线条宽度 | 否 | Number | 10 | |
baseColor | 底图线条颜色 | 否 | String | '#e5e9f2' | |
lineWidth | 进度条的宽度 | 否 | Number | 8 | |
fillet | 进度条是否开启圆角 | 否 | Boolean | true | |
withGradient | 进度条是否开启渐变色 | 否 | Boolean | true | |
graColor | 渐变颜色组 | 否 | Array<String> | ['#13CDE3', '#3B77E3'] | |
lineColor | 不渐变时进度条的颜色 | 否 | String | '#42cccc' | |
isShowPoint | 是否开启进度点 | 否 | Boolean | false | |
pointRadius | 进度点半径 | 否 | Number | 8 | |
pointColor | 进度点颜色 | 否 | String | '#3B77E3' | |
label | 中心文字 | 否 | String | | |
fontSize | 文字大小 | 否 | Number | 16 | |
fontColor | 文字颜色 | 否 | String | '#42cccc' | |
fontWeight | 文字粗细 | 否 | Number | 400 | 100、200、300...900 |
rate | 进度条速度 | 否 | String | 'normal' | 'fast'、'normal'、'slow'、'snail' |

