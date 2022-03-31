<template>
  <canvas ref="plum" :width="WIDTH" :height="HEIGHT" border />
</template>
<script setup lang="ts">
import type { Nullable } from 'vitest'

const plum = $ref<Nullable<HTMLCanvasElement>>()
const ctx = $computed(() => plum!.getContext('2d')!)

const WIDTH = 600
const HEIGHT = 1000

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

  lineTo(end: Point, color: string) {
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

  draw(color = '#000') {
    this.lineTo(this.getEndPoint(), color)
  }
}

function init() {
  const start = new Line(new Point(WIDTH / 2, HEIGHT), 100, -Math.PI / 2)
  start.draw('red')

  new Line(start.getEndPoint(), 100, start.theta - 0.5).draw('blue')
}

onMounted(() => {
  init()
})
</script>
