<script setup lang="ts">
const el = $ref<HTMLCanvasElement>();
const ctx = $computed(() => el!.getContext("2d")!);
const WIDTH = 600;
const HEIGHT = 600;
interface Point {
  x: number;
  y: number;
}
interface Branch {
  startPoint: Point;
  length: number;
  theta: number;
}
function init() {
  ctx.strokeStyle = "black";
  step({
    startPoint: { x: WIDTH / 2, y: HEIGHT },
    length: 20,
    theta: -Math.PI / 2,
  });
}

const pendingTasks: Function[] = [];
function step(b: Branch, depth) {
  const endPoint = getEndPoint(b);
  drawBranch(b);
  if (depth < 1 || Math.random() < 0.55) {
    pendingTasks.push(() =>
      step({
        startPoint: endPoint,
        length: b.length + Math.random() * 10 - 5,
        theta: b.theta - 0.4 * Math.random(),
      }, depth + 1)
    );
  }
  if (depth < 1 || Math.random() < 0.5) {
    pendingTasks.push(() =>
      step({
        startPoint: endPoint,
        length: b.length + Math.random() * 10 - 5,
        theta: b.theta + 0.4 * Math.random(),
      }, depth + 1)
    );
  }
}
function frame() {
  const tasks = [...pendingTasks];
  pendingTasks.length = 0;
  tasks.forEach((task) => task());
}
let frameCount = 0;

function startFrame() {
  requestAnimationFrame(() => {
    frameCount += 1;
    if (frameCount %10 === 0) {
      frame();
    }
    startFrame();
  });
}
startFrame();
function lineTo(p1: Point, p2: Point) {
  ctx.beginPath();
  ctx.moveTo(p1.x, p1.y);
  ctx.lineTo(p2.x, p2.y);
  ctx.stroke();
}
function getEndPoint(b: branch) {
  return {
    x: b.startPoint.x + b.length * Math.cos(b.theta),
    y: b.startPoint.y + b.length * Math.sin(b.theta),
  };
}
function drawBranch(l: Line) {
  lineTo(l.startPoint, getEndPoint(l));
}
onMounted(() => {
  init();
});
</script>

<template>
  <canvas ref="el" width="600" height="600" border />
</template>
