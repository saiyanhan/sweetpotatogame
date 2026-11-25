<script setup>
import { ref } from 'vue'

// 畫面切換狀態：'home' | 'game' | 'result' | 'gallery'
const currentStep = ref('home')

// 四種地瓜商品
const items = [
  { id: 'red', name: '烤紅薯', icon: '🍠' },
  { id: 'white', name: '烤白薯', icon: '🍠' },
  { id: 'butter', name: '奶油地瓜', icon: '🧈🍠' },
  { id: 'purple', name: '紫薯', icon: '🍠' }
]

// 參加者名單與頭像（手邊有圖後可換成 './images/characters/xxx.png'）
const characters = [
  { name: '小貓咪', avatar: 'https://api.dicebear.com/7.x/bottts/svg?seed=cat1' },
  { name: '柴犬君', avatar: 'https://api.dicebear.com/7.x/bottts/svg?seed=dog2' },
  { name: '兔兔店員', avatar: 'https://api.dicebear.com/7.x/bottts/svg?seed=rabbit3' },
  { name: '熊熊客人', avatar: 'https://api.dicebear.com/7.x/bottts/svg?seed=bear4' }
]

// 遊戲狀態變數
const currentRound = ref(1)
const score = ref(0)
const feedbackMessage = ref('')
const feedbackType = ref('') // 'correct' | 'wrong'
const currentOrder = ref(items[0])
const currentCharacter = ref(characters[0])
const isLocked = ref(false)

// 隨機出題
const generateQuestion = () => {
  currentCharacter.value = characters[Math.floor(Math.random() * characters.length)]
  currentOrder.value = items[Math.floor(Math.random() * items.length)]
}

// 開始遊戲
const startGame = () => {
  currentRound.value = 1
  score.value = 0
  feedbackMessage.value = ''
  isLocked.value = false
  generateQuestion()
  currentStep.value = 'game'
}

// 玩家作答
const handleSelect = (selectedItem) => {
  if (isLocked.value) return
  isLocked.value = true

  if (selectedItem.id === currentOrder.value.id) {
    score.value += 1
    feedbackMessage.value = 'ありがとうございます！'
    feedbackType.value = 'correct'
  } else {
    score.value -= 1
    feedbackMessage.value = 'あれ？注文と違います！'
    feedbackType.value = 'wrong'
  }

  // 1 秒後切換至下一題或結算
  setTimeout(() => {
    feedbackMessage.value = ''
    if (currentRound.value >= 10) {
      currentStep.value = 'result'
    } else {
      currentRound.value += 1
      generateQuestion()
      isLocked.value = false
    }
  }, 1000)
}
</script>

<template>
  <main class="game-container">
    <!-- ① 活動首頁 -->
    <div v-if="currentStep === 'home'" class="view view-home">
      <div class="stall-badge">冬日溫暖限定</div>
      <h1 class="main-title">焼き芋屋さんへようこそ！</h1>
      <p class="sub-text">暖呼呼的烤地瓜出爐囉！</p>

      <div class="wood-actions">
        <button class="wood-btn primary" @click="startGame">
          <span class="btn-icon">🍠</span> 開始遊戲
        </button>
        <button class="wood-btn secondary" @click="currentStep = 'gallery'">
          <span class="btn-icon">🖼️</span> 只想參觀
        </button>
      </div>
    </div>

    <!-- ② 遊戲畫面 -->
    <div v-else-if="currentStep === 'game'" class="view view-game">
      <header class="game-header">
        <span class="badge round-badge">第 {{ currentRound }} / 10 題</span>
        <span class="badge score-badge">累積分數：{{ score }}</span>
      </header>

      <div class="character-card">
        <img :src="currentCharacter.avatar" :alt="currentCharacter.name" class="character-img" />
        <div class="speech-bubble">
          <span class="char-name">{{ currentCharacter.name }}：</span>
          <p>請給我一份 <strong>{{ currentOrder.name }}</strong>！</p>
        </div>
      </div>

      <div class="feedback-area">
        <transition name="fade">
          <div v-if="feedbackMessage" :class="['feedback-toast', feedbackType]">
            {{ feedbackMessage }}
          </div>
        </transition>
      </div>

      <div class="options-grid">
        <button
          v-for="item in items"
          :key="item.id"
          class="option-card"
          :disabled="isLocked"
          @click="handleSelect(item)"
        >
          <span class="item-icon">{{ item.icon }}</span>
          <span class="item-name">{{ item.name }}</span>
        </button>
      </div>
    </div>

    <!-- ③ 分數結算畫面 -->
    <div v-else-if="currentStep === 'result'" class="view view-result">
      <div class="result-box">
        <h2 class="result-title">おつかれさまでした！</h2>
        <div class="score-display">
          <span class="label">本次得分</span>
          <span class="number">{{ score }} <small>/ 10 分</small></span>
        </div>
        <button class="wood-btn primary" @click="currentStep = 'gallery'">
          查看全員合繪
        </button>
      </div>
    </div>

    <!-- ④ 合圖展示畫面 -->
    <div v-else-if="currentStep === 'gallery'" class="view view-gallery">
      <h2 class="gallery-title">合圖成果展示</h2>
      
      <div class="art-frame">
        <img 
          src="https://placehold.co/600x400/8c4b2d/ffffff?text=全員合繪大圖" 
          alt="全員合繪大圖" 
          class="full-art-img" 
        />
      </div>

      <div class="credit-list">
        <h3>參加名單</h3>
        <p>繪師 A、繪師 B、繪師 C、繪師 D</p>
      </div>

      <button class="wood-btn secondary back-btn" @click="currentStep = 'home'">
        回到首頁
      </button>
    </div>
  </main>
</template>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  flex: 1;
  padding: 24px;
}

.view {
  display: flex;
  flex-direction: column;
  flex: 1;
}

/* 木質按鈕風格 */
.wood-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
  padding: 16px;
  font-size: 1.15rem;
  font-weight: bold;
  border: 3px solid #5a3825;
  border-radius: 12px;
  cursor: pointer;
  box-shadow: 0 4px 0 #5a3825;
  transition: transform 0.1s, box-shadow 0.1s;
}

.wood-btn:active {
  transform: translateY(4px);
  box-shadow: 0 0 0 #5a3825;
}

.wood-btn.primary {
  background-color: #d97443;
  color: #fffdf9;
}

.wood-btn.secondary {
  background-color: #f7e1b5;
  color: #5a3825;
}

/* 首頁樣式 */
.view-home {
  justify-content: center;
  align-items: center;
  text-align: center;
  gap: 20px;
}

.stall-badge {
  background-color: #7b321c;
  color: #fff;
  padding: 4px 14px;
  border-radius: 20px;
  font-size: 0.9rem;
}

.main-title {
  font-size: 1.6rem;
  color: #522719;
}

.sub-text {
  color: #7d5e4b;
  margin-bottom: 24px;
}

.wood-actions {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
  max-width: 280px;
}

/* 遊戲畫面樣式 */
.game-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 24px;
}

.badge {
  background: #f1dfcf;
  padding: 8px 16px;
  border-radius: 20px;
  font-weight: bold;
  color: #633f2c;
}

.character-card {
  display: flex;
  align-items: center;
  gap: 16px;
  background: #ffffff;
  padding: 16px;
  border-radius: 16px;
  border: 2px solid #ebd9cb;
  margin-bottom: 16px;
}

.character-img {
  width: 72px;
  height: 72px;
  border-radius: 50%;
  background: #fdf5ea;
  border: 2px solid #d9b89d;
}

.speech-bubble {
  flex: 1;
}

.speech-bubble .char-name {
  font-size: 0.85rem;
  color: #8c6a58;
}

.speech-bubble p {
  font-size: 1.1rem;
  color: #4a2810;
  margin-top: 4px;
}

.feedback-area {
  min-height: 48px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 16px;
}

.feedback-toast {
  padding: 10px 20px;
  border-radius: 8px;
  font-weight: bold;
}

.feedback-toast.correct {
  background-color: #e2f5dc;
  color: #2b7a1d;
  border: 1px solid #a8e09f;
}

.feedback-toast.wrong {
  background-color: #fde8e8;
  color: #b92b27;
  border: 1px solid #f8b4b4;
}

.options-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-top: auto;
}

.option-card {
  background: #fff;
  border: 2px solid #d9b89d;
  border-radius: 12px;
  padding: 20px 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  transition: transform 0.1s, background-color 0.2s;
}

.option-card:hover {
  background-color: #fff9f2;
}

.option-card:active {
  transform: scale(0.96);
}

.item-icon {
  font-size: 2rem;
}

.item-name {
  font-size: 1rem;
  font-weight: bold;
  color: #522719;
}

/* 結算樣式 */
.view-result {
  justify-content: center;
  align-items: center;
}

.result-box {
  background: #fff;
  border: 3px solid #5a3825;
  border-radius: 20px;
  padding: 32px 24px;
  text-align: center;
  width: 100%;
}

.result-title {
  color: #522719;
  margin-bottom: 24px;
}

.score-display {
  display: flex;
  flex-direction: column;
  margin-bottom: 28px;
}

.score-display .label {
  font-size: 1rem;
  color: #8c6a58;
}

.score-display .number {
  font-size: 2.8rem;
  font-weight: bold;
  color: #d97443;
}

/* 合圖樣式 */
.view-gallery {
  align-items: center;
  text-align: center;
  gap: 16px;
}

.gallery-title {
  color: #522719;
}

.art-frame {
  width: 100%;
  border-radius: 12px;
  overflow: hidden;
  border: 3px solid #5a3825;
}

.full-art-img {
  width: 100%;
  display: block;
}

.credit-list {
  background: #fdf5ea;
  padding: 16px;
  border-radius: 12px;
  width: 100%;
}

.credit-list h3 {
  font-size: 1rem;
  color: #522719;
  margin-bottom: 8px;
}

.back-btn {
  margin-top: auto;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>