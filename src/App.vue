<template>
  <div class="stage" :class="{ dark }">
    <canvas ref="plum" :width="WIDTH" :height="HEIGHT" />
  </div>
</template>
<script setup lang="ts">
import type { Nullable } from 'vitest'

class Point {
  x: number
  y: number
  constructor(x: number, y: number) {
    this.x = x
    this.y = y
  }
}

class Line {
  start: Point
  end: Nullable<Point>
  length: number
  theta: number
  constructor(start: Point, length: number, theta: number) {
    this.start = start
    this.length = length
    this.theta = theta
    this.end = null
  }

  lineTo(ctx: CanvasRenderingContext2D, end: Point, color: string) {
    ctx.strokeStyle = color
    ctx.beginPath()
    ctx.moveTo(this.start.x, this.start.y)
    ctx.lineTo(end.x, end.y)
    ctx.stroke()
  }

  getEndPoint() {
    if (this.end) return this.end

    const { start, length, theta } = this
    const { x, y } = start
    const { cos, sin } = Math
    const endX = x + length * cos(theta)
    const endY = y + length * sin(theta)

    this.end = new Point(
      endX,
      endY,
    )

    return this.end
  }

  draw(ctx: CanvasRenderingContext2D, color = branchColor) {
    this.lineTo(ctx, this.getEndPoint(), color)
  }
}

class TaskQueue {
  tasks: Array<() => void>
  constructor() {
    this.tasks = []
  }

  add(task: () => void) {
    this.tasks.push(task)
  }

  run() {
    const tasks = [...this.tasks]

    this.tasks = []
    tasks.forEach(task => task())
  }
}

const themeParam = new URLSearchParams(location.search).get('theme')
const dark = themeParam
  ? themeParam === 'dark'
  : window.matchMedia('(prefers-color-scheme: dark)').matches
const branchColor = dark ? '#ededed' : '#000'

const plum = $ref<Nullable<HTMLCanvasElement>>()
const ctx = $computed(() => plum!.getContext('2d')!)
const taskQueue = $ref(new TaskQueue())

let frameCount = $ref(0)

const WIDTH = 600
const HEIGHT = 600

function inBoundary(end: Point, line: Line) {
  return end.y > -line.length && end.y < HEIGHT + line.length && end.x < WIDTH + line.length && end.x > 0 - line.length
}

function step(line: Line) {
  const end = line.getEndPoint()

  line.draw(ctx)

  if (Math.random() > 0.3 && inBoundary(end, line))
    taskQueue.add(() => step(new Line(end, line.length, line.theta - Math.random() * 0.3)))

  if (Math.random() > 0.3 && inBoundary(end, line))
    taskQueue.add(() => step(new Line(end, line.length, line.theta + Math.random() * 0.3)))
}

function runFrame() {
  requestAnimationFrame(() => {
    frameCount += 1

    if (frameCount % 3 === 0)
      taskQueue.run()

    runFrame()
  })
}

function init(branchCount: number, length = 40) {
  // 底部居中、角度 -90°±0.2 散开：随机起点/随机角度经常横着飞出画布，
  // 作为展柜 demo 需要稳定长出「树形」
  new Array(branchCount).fill(0).forEach((_, i) => {
    const offset = (i - (branchCount - 1) / 2) * 24
    const theta = -Math.PI / 2 + (Math.random() - 0.5) * 0.4
    step(new Line(new Point(WIDTH / 2 + offset, HEIGHT), length, theta))
  })

  runFrame()
}

onMounted(() => {
  init(5, 40)
})
</script>
<style scoped lang="scss">
.stage {
  width: 100vw;
  height: 100vh;
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #ffffff;
  &.dark {
    background: #0a0a0a;
    canvas {
      border-color: #333;
    }
  }
}
canvas {
  width: min(88vmin, 880px);
  height: min(88vmin, 880px);
  border: 1px solid #000;
}
</style>
