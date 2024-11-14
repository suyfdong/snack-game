<template>
  <canvas ref="canvas" width="400" height="400"></canvas>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['score-change', 'high-score-change'])

// 定义游戏参数
const GRID_SIZE = 20
const SNAKE_SPEED = 200
const BOARD_SIZE = 20 // 20x20的网格
// 在现有的游戏参数后面添加（在 GRID_SIZE, BOARD_SIZE 等常量后面）
const SPEEDS = {
  easy: 300,    // 简单：较慢
  medium: 200,  // 中等：标准速度
  hard: 100     // 困难：较快
}

let currentSpeed = SPEEDS.medium // 默认中等速度

// 创建canvas引用
const canvas = ref<HTMLCanvasElement | null>(null)
let ctx: CanvasRenderingContext2D | null = null

// 游戏状态
let snake = [{ x: 5, y: 5 }]
let direction = 'right'
let food = { x: 10, y: 10 }
let gameInterval: number | null = null
let score = 0

// 初始化游戏
onMounted(() => {
  if (canvas.value) {
    ctx = canvas.value.getContext('2d')
    document.addEventListener('keydown', handleKeyPress)
    generateFood()
    // 初始绘制
    clearBoard()
    drawSnake()
    drawFood()
  }
})

// 清理事件监听
onUnmounted(() => {
  document.removeEventListener('keydown', handleKeyPress)
  if (gameInterval) clearInterval(gameInterval)
})

// 生成食物
function generateFood() {
  while (true) {
    food = {
      x: Math.floor(Math.random() * BOARD_SIZE),
      y: Math.floor(Math.random() * BOARD_SIZE)
    }
    if (!snake.some(segment => segment.x === food.x && segment.y === food.y)) {
      break
    }
  }
}
// 添加设置难度的方法（在 generateFood 函数后面添加）
function setDifficulty(level: keyof typeof SPEEDS) {
  currentSpeed = SPEEDS[level]
  // 如果游戏正在运行，需要重新启动以应用新速度
  if (gameInterval) {
    clearInterval(gameInterval)
    gameInterval = setInterval(gameLoop, currentSpeed)
  }
}

// 检查是否吃到食物
function checkFood(head: {x: number, y: number}) {
  if (head.x === food.x && head.y === food.y) {
    generateFood()
    score += 10
    emit('score-change', score)
    return true
  }
  return false
}

// 检查碰撞
function checkCollision(head: {x: number, y: number}) {
  if (head.x < 0 || head.x >= BOARD_SIZE || head.y < 0 || head.y >= BOARD_SIZE) {
    return true
  }
  return snake.some(segment => segment.x === head.x && segment.y === head.y)
}

// 游戏结束
function gameOver() {
  pauseGame()
  alert(`游戏结束！\n最终得分：${score}`)
  // 检查是否创造新高分
  const highScore = Number(localStorage.getItem('snakeHighScore')) || 0
  if (score > highScore) {
    localStorage.setItem('snakeHighScore', String(score))
    emit('high-score-change', score)
  }
  // 重置游戏
  snake = [{ x: 5, y: 5 }]
  direction = 'right'
  score = 0
  emit('score-change', score)
  generateFood()
  // 重新绘制
  clearBoard()
  drawSnake()
  drawFood()
}

// 移动蛇
function moveSnake() {
  const head = { ...snake[0] }
  
  switch (direction) {
    case 'up': head.y--; break
    case 'down': head.y++; break
    case 'left': head.x--; break
    case 'right': head.x++; break
  }

  if (checkCollision(head)) {
    gameOver()
    return
  }

  snake.unshift(head)
  if (!checkFood(head)) {
    snake.pop()
  }
}

// 游戏循环
function gameLoop() {
  if (!ctx) return
  clearBoard()
  moveSnake()
  drawSnake()
  drawFood()
}

// 开始游戏
function startGame() {
  if (!gameInterval) {
    clearBoard()
    drawSnake()
    drawFood()
    gameInterval = setInterval(gameLoop, SNAKE_SPEED)
  }
}

// 暂停游戏
function pauseGame() {
  if (gameInterval) {
    clearInterval(gameInterval)
    gameInterval = null
  }
  // 确保暂停时画面保持不变
  clearBoard()
  drawSnake()
  drawFood()
}

function clearBoard() {
  if (!ctx || !canvas.value) return
  ctx.fillStyle = '#f0f0f0'
  ctx.fillRect(0, 0, canvas.value.width, canvas.value.height)
}

function drawSnake() {
  if (!ctx) return
  ctx.fillStyle = '#4CAF50'
  snake.forEach(segment => {
    ctx!.fillRect(
      segment.x * GRID_SIZE,
      segment.y * GRID_SIZE,
      GRID_SIZE - 2,
      GRID_SIZE - 2
    )
  })
}

function drawFood() {
  if (!ctx) return
  ctx.fillStyle = '#FF5722'
  ctx.fillRect(
    food.x * GRID_SIZE,
    food.y * GRID_SIZE,
    GRID_SIZE - 2,
    GRID_SIZE - 2
  )
}

function handleKeyPress(event: KeyboardEvent) {
  switch (event.key) {
    case 'ArrowUp':
      if (direction !== 'down') direction = 'up'
      break
    case 'ArrowDown':
      if (direction !== 'up') direction = 'down'
      break
    case 'ArrowLeft':
      if (direction !== 'right') direction = 'left'
      break
    case 'ArrowRight':
      if (direction !== 'left') direction = 'right'
      break
  }
}

// 暴露方法给父组件
defineExpose({
  startGame,
  pauseGame,
  setDifficulty,
  handleKeyPress  // 添加这一行
})
</script>

<style scoped>
canvas {
  background-color: #f0f0f0;
}
</style>