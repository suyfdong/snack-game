<template>
  <div class="game-container">
    <h1>贪吃蛇游戏</h1>
    <div class="score-board">
      <div class="score">当前分数：{{ score }}</div>
      <div class="high-score">最高分数：{{ highScore }}</div>
    </div>
    
    <!-- 难度选择 -->
    <div class="difficulty-selector">
      <span>难度：</span>
      <select v-model="difficulty" @change="changeDifficulty" :disabled="isPlaying">
        <option value="easy">简单</option>
        <option value="medium">中等</option>
        <option value="hard">困难</option>
      </select>
    </div>

    <div class="game-board" ref="gameBoard">
      <GameBoard 
        @score-change="updateScore" 
        @high-score-change="updateHighScore"
        ref="gameRef"
      />
    </div>

    <!-- 移动端控制按钮 -->
    <div class="mobile-controls">
      <div class="control-row">
        <button @click="handleDirection('ArrowUp')" class="direction-btn up">↑</button>
      </div>
      <div class="control-row">
        <button @click="handleDirection('ArrowLeft')" class="direction-btn">←</button>
        <button @click="handleDirection('ArrowDown')" class="direction-btn">↓</button>
        <button @click="handleDirection('ArrowRight')" class="direction-btn">→</button>
      </div>
    </div>

    <div class="controls">
      <button @click="toggleGame" class="control-btn">
        {{ isPlaying ? '暂停' : '开始' }}
      </button>
    </div>

    <div class="instructions">
      <h2>游戏说明：</h2>
      <ul>
        <li>使用键盘方向键 ↑↓←→ 或屏幕按钮控制蛇的移动</li>
        <li>吃到食物（橙色方块）可得10分</li>
        <li>撞到墙壁或自己会结束游戏</li>
        <li>简单：蛇速较慢，适合新手</li>
        <li>中等：标准速度</li>
        <li>困难：高速移动，考验反应</li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import GameBoard from './components/GameBoard.vue'

const score = ref(0)
const highScore = ref(Number(localStorage.getItem('snakeHighScore')) || 0)
const isPlaying = ref(false)
const gameRef = ref()
const difficulty = ref('medium') // 默认中等难度

const handleDirection = (direction: string) => {
  if (gameRef.value) {
    gameRef.value.handleKeyPress({ key: direction })
  }
}

const updateScore = (newScore: number) => {
  score.value = newScore
  if (newScore > highScore.value) {
    updateHighScore(newScore)
  }
}

const updateHighScore = (newHighScore: number) => {
  highScore.value = newHighScore
  localStorage.setItem('snakeHighScore', String(newHighScore))
}

const changeDifficulty = () => {
  if (gameRef.value) {
    gameRef.value.setDifficulty(difficulty.value)
  }
}

const toggleGame = () => {
  if (gameRef.value) {
    isPlaying.value = !isPlaying.value
    if (isPlaying.value) {
      gameRef.value.startGame()
    } else {
      gameRef.value.pauseGame()
    }
  }
}
</script>

<style scoped>
.game-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

.score-board {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin: 20px 0;
  font-size: 20px;
}

.game-board {
  margin: 20px auto;
}

.controls {
  margin: 20px 0;
}

.control-btn {
  padding: 10px 20px;
  font-size: 18px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.control-btn:hover {
  background-color: #3aa876;
}

.instructions {
  margin-top: 30px;
  text-align: left;
}

.instructions ul {
  list-style-type: none;
  padding: 0;
}

.instructions li {
  margin: 10px 0;
}

/* 难度选择器样式 */
.difficulty-selector {
  margin: 10px 0;
  font-size: 18px;
}

.difficulty-selector select {
  margin-left: 10px;
  padding: 5px 10px;
  font-size: 16px;
  border: 2px solid #42b983;
  border-radius: 5px;
  background-color: white;
  color: #2c3e50;
  cursor: pointer;
}

.difficulty-selector select:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* 移动端控制按钮样式 */
.mobile-controls {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
  margin: 20px 0;
}

.control-row {
  display: flex;
  gap: 10px;
}

.direction-btn {
  width: 60px;
  height: 60px;
  font-size: 24px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  user-select: none;
  -webkit-tap-highlight-color: transparent;
}

.direction-btn:active {
  background-color: #3aa876;
}
</style>