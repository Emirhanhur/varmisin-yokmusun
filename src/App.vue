<template>
  <div class="app">
    <div class="container-fluid ">
      <div class="row">
        <div class="col-2 col-auto">
          <div class="value-list">
            <div v-for="box in firstFifteenValues" :key="box.number" class="value-item" :class="{ 'revealed': box.revealed }">
              {{ formatNumber(box.value) }} 
            </div>
          </div>
        </div>

        
        <div class="col-8 ">
          <h1 class="main-title">Var Mısın Yok Musun?</h1>

          <div v-if="isPortraitOnMobile" class="orientation-warning">
            <div class="orientation-message">
              <strong>Lütfen cihazınızı yatay (landscape) moda çevirin.<br>Oyun dikey modda oynanamaz.</strong>
            </div>
          </div>
          <div v-else>
            <div v-if="!gameStarted" class="setup beautiful-setup">
              <div class="setup-card">
                <h2 class="setup-title">Oyuncu İsimleri</h2>
                <p class="setup-desc">En az 2 kişiyle oynanır. Kişi ekle butonuyla istediğin kadar oyuncu ekleyebilirsin.
                </p>
                <transition-group name="fade-move" tag="div" class="player-inputs beautiful-inputs">
                  <div v-for="(player, index) in players" :key="index" class="input-group player-card">
                    <div class="avatar">
                    </div>
                    <input v-model="players[index]" :placeholder="`${index + 1}. Oyuncu Adı`" maxlength="16" />
                  </div>
                </transition-group>
                <div class="button-group beautiful-btn-group">
                  <button @click="addPlayer" class="add-player-btn beautiful-add-btn">
                    <span class="plus-icon">+</span> Kişi Ekle
                  </button>
                  <button @click="startGame" :disabled="!canStartGame" class="start-btn">Oyunu Başlat</button>
                </div>
              </div>
            </div>

            <div v-else-if="!gameOver" class="game">
              <div class="game-info">
                <h2>Sıra: {{ currentPlayer }}</h2>
              </div>

              <div class="boxes">
                <div v-for="box in sortedBoxesByNumber" :key="box.number" class="box"
                  :class="{ 'selected': box.selected, 'revealed': box.revealed }" @click="selectBox(box)">
                  <div class="box-front">
                    <span>{{ box.number }}</span>
                  </div>
                  <div class="box-back">
                    <span>{{ formatNumber(box.value) }} TL</span>
                  </div>
                </div>
              </div>
            </div>

            <div v-else class="results">
              <h2>BUGUNUN ŞANSLI ŞAHSİYETİ :</h2>
              <div class="results-list">
                <div v-for="(result, index) in sortedResults" :key="index" class="result-item">
                  <span class="rank">{{ index + 1 }}.</span>
                  <span class="player-name">{{ result.player }}</span>
                  <span class="box-value">{{ formatNumber(result.value) }} TL</span>
                </div>
              </div>
              <button @click="resetGame" class="reset-btn ">Yeni Oyun</button>
            </div>

            <!-- Box Modal -->
            <BoxModal v-if="showModal" :opened="modalOpened" :amount="modalAmount" @close="closeModal" />
          </div>
        </div>
        
        <div class="col-2 col-auto">
          <div class="value-list">
            <div v-for="box in lastFifteenValues" :key="box.number" class="value-item" :class="{ 'revealed': box.revealed }">
              {{ formatNumber(box.value) }} 
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import BoxModal from './components/BoxModal.vue';

export default {
  name: 'App',
  components: { BoxModal },
  data() {
    return {
      players: ['', ''],
      gameStarted: false,
      gameOver: false,
      currentPlayer: '',
      boxes: [],
      selectedBox: null,
      playerResults: [],
      // Modal state
      showModal: false,
      modalOpened: false,
      modalAmount: null,
      modalBox: null,
      pendingBox: null, // for game logic after modal
      isPortraitOnMobile: false
    }
  },
  computed: {
    canStartGame() {
      return this.players.every(player => player.trim()) && this.players.length >= 2
    },
    sortedResults() {
      return [...this.playerResults].sort((a, b) => b.value - a.value)
    },
    sortedBoxesByNumber() {
      return [...this.boxes].sort((a, b) => a.number - b.number)
    },
    sortedBoxesByValue() {
      return [...this.boxes].sort((a, b) => a.value - b.value)
    },
    firstFifteenValues() {
      return this.sortedBoxesByValue.slice(0, 15)
    },
    lastFifteenValues() {
      return this.sortedBoxesByValue.slice(15)
    }
  },
  mounted() {
    this.checkOrientation();
    window.addEventListener('resize', this.checkOrientation);
    window.addEventListener('orientationchange', this.checkOrientation);
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.checkOrientation);
    window.removeEventListener('orientationchange', this.checkOrientation);
  },
  methods: {
    formatNumber(number) {
      if (number >= 1000) {
        return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
      }
      return number;
    },
    addPlayer() {
      this.players.push('')
    },
    startGame() {
      this.gameStarted = true
      this.gameOver = false
      // Shuffle players array
      this.players = this.players.sort(() => Math.random() - 0.5)
      this.currentPlayer = this.players[0]
      this.playerResults = []
      this.initializeBoxes()
    },
    initializeBoxes() {
      const prizeValues = [
        1, 5, 10, 25, 50, 75, 100, 200, 300, 400, 500, 750, 1000,
        2500, 5000, 7500, 10000, 15000, 20000, 25000, 30000, 40000,
        50000, 60000, 75000, 100000, 150000, 200000, 250000, 500000
      ];

      // Sayıları karıştır ve ilk 30'u al
      const shuffledPrizes = [...prizeValues]
        .sort(() => Math.random() - 0.5)
        .slice(0, 30);

      // Kutuları oluştur
      this.boxes = shuffledPrizes.map((value, index) => ({
        number: index + 1,
        value: value,
        selected: false,
        revealed: false
      }));
    },
    selectBox(box) {
      if (box.revealed || box.selected || this.showModal) return
      this.showModal = true
      this.modalOpened = false
      this.modalBox = box
      this.modalAmount = box.value
      this.pendingBox = box
      // Show closed box first, then open after 1s
      setTimeout(() => {
        this.modalOpened = true
      }, 1000)
    },
    closeModal() {
      // Reveal the box and continue game logic
      const box = this.pendingBox
      if (box) {
        box.revealed = true
        box.selected = false
        this.selectedBox = null
        // Oyuncunun seçtiği kutuyu kaydet
        this.playerResults.push({
          player: this.currentPlayer,
          value: box.value
        })
        // Sırayı diğer oyuncuya geçir
        const currentIndex = this.players.indexOf(this.currentPlayer)
        const nextIndex = (currentIndex + 1) % this.players.length
        this.currentPlayer = this.players[nextIndex]
        // Tüm oyuncular seçim yaptıysa oyunu bitir
        if (this.playerResults.length === this.players.length) {
          this.gameOver = true
        }
      }
      this.showModal = false
      this.modalOpened = false
      this.modalBox = null
      this.modalAmount = null
      this.pendingBox = null
    },
    resetGame() {
      this.gameStarted = false
      this.gameOver = false
      this.players = ['', '']
      this.boxes = []
      this.playerResults = []
    },
    checkOrientation() {
      const isMobile = window.innerWidth <= 600;
      const isPortrait = window.innerHeight > window.innerWidth;
      this.isPortraitOnMobile = isMobile && isPortrait;
    }
  }
}
</script>

<style>
.app {

  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Arial, sans-serif;
  background:url(./assets/background.jpg);
  min-height: 100vh;
}

.main-title {
  background: rgba(0, 0, 0, 0.5);
  text-align: center;
  color: rgb(255, 255, 255);
  font-size: 2.7rem;
  font-weight: 800;
  letter-spacing: 1px;
  margin-bottom: 32px;
  text-shadow: 0 2px 8px rgba(69, 162, 255, 0.08);
}

.beautiful-setup {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: 60vh;
}

.setup-card {
  background: #fff;
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(44, 62, 80, 0.10), 0 1.5px 4px rgba(44, 62, 80, 0.08);
  padding: 40px 32px 32px 32px;
  min-width: 340px;
  max-width: 420px;
  width: 100%;
  margin-top: 32px;
  animation: pop-in 0.7s cubic-bezier(.68, -0.55, .27, 1.55);
}

@keyframes pop-in {
  0% {
    transform: scale(0.8) translateY(40px);
    opacity: 0;
  }

  100% {
    transform: scale(1) translateY(0);
    opacity: 1;
  }
}

.setup-title {
  font-size: 1.7rem;
  font-weight: 700;
  color: rgb(0, 0, 0);
  margin-bottom: 8px;
  text-align: center;
}

.setup-desc {
  color: #64748b;
  font-size: 1.05rem;
  margin-bottom: 24px;
  text-align: center;
}

.beautiful-inputs {
  display: flex;
  flex-direction: column;
  gap: 18px;
  margin: 18px 0 28px 0;
}

.player-card {
  display: flex;
  align-items: center;
  background: #f1f5f9;
  border-radius: 14px;
  box-shadow: 0 2px 8px rgba(44, 62, 80, 0.04);
  padding: 10px 16px;
  transition: box-shadow 0.2s;
  position: relative;
}

.player-card:focus-within {
  box-shadow: 0 4px 16px rgba(44, 62, 80, 0.10);
}

.avatar {
  width: 38px;
  height: 38px;
  border-radius: 50%;
  background: linear-gradient(135deg, #6366f1 60%, #818cf8 100%);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.3rem;
  font-weight: 700;
  margin-right: 14px;
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.10);
}

.player-card input {
  flex: 1;
  padding: 10px 14px;
  font-size: 1.1rem;
  border: none;
  border-radius: 8px;
  background: #fff;
  box-shadow: 0 1px 2px rgba(44, 62, 80, 0.04);
  outline: none;
  transition: box-shadow 0.2s;
}

.player-card input:focus {
  box-shadow: 0 2px 8px #6366f1aa;
}

.beautiful-btn-group {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-top: 18px;
}

.beautiful-add-btn {
  background: linear-gradient(90deg, #6366f1 60%, #818cf8 100%);
  color: #fff;
  font-weight: 600;
  border: none;
  border-radius: 8px;
  padding: 10px 22px;
  font-size: 1.1rem;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.10);
  display: flex;
  align-items: center;
  gap: 8px;
  transition: background 0.2s, box-shadow 0.2s, transform 0.15s;
}

.beautiful-add-btn:hover {
  background: linear-gradient(90deg, #818cf8 60%, #6366f1 100%);
  box-shadow: 0 4px 16px rgba(99, 102, 241, 0.18);
  transform: translateY(-2px) scale(1.04);
}

.plus-icon {
  font-size: 1.3rem;
  font-weight: 900;
  margin-right: 2px;
}

.start-btn {
  background: linear-gradient(90deg, #22c55e 60%, #4ade80 100%);
  color: #fff;
  font-weight: 700;
  border: none;
  border-radius: 8px;
  padding: 10px 22px;
  font-size: 1.1rem;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(34, 197, 94, 0.10);
  transition: background 0.2s, box-shadow 0.2s, transform 0.15s;
}

.start-btn:disabled {
  background: #cbd5e1;
  color: #64748b;
  cursor: not-allowed;
  box-shadow: none;
}

.start-btn:not(:disabled):hover {
  background: linear-gradient(90deg, #4ade80 60%, #22c55e 100%);
  box-shadow: 0 4px 16px rgba(34, 197, 94, 0.18);
  transform: translateY(-2px) scale(1.04);
}

.fade-move-enter-active,
.fade-move-leave-active {
  transition: all 0.4s cubic-bezier(.68, -0.55, .27, 1.55);
}

.fade-move-enter-from,
.fade-move-leave-to {
  opacity: 0;
  transform: translateY(20px) scale(0.95);
}

@media (max-width: 600px) {
  .setup-card {
    padding: 18px 6vw 18px 6vw;
    min-width: unset;
    max-width: 98vw;
  }

  .main-title {
    font-size: 1.8rem;
  }

  .setup-title {
    font-size: 1.4rem;
  }

  .setup-desc {
    font-size: 0.9rem;
  }

  .player-card input {
    font-size: 0.9rem;
  }

  .beautiful-add-btn, .start-btn {
    font-size: 0.9rem;
  }

  .box-front span, .box-back span {
    font-size: 18px;
  }

  .result-item {
    font-size: 0.9rem;
  }

  .value-item {
    font-size: 0.9rem;
  }
}

.game-info {
  text-align: center;
  margin-bottom: 20px;
  color: white;
  background-color: rgba(0, 0, 0, 0.5);
}

.boxes {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 15px;
  perspective: 2000px;
  padding: 20px;
  min-height:500px !important;
  
}

.box {
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 20px;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  transform-style: preserve-3d;
  position: relative;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
  
}

.box:hover {
  transform: scale(1.05) translateZ(20px);
  box-shadow: 0 15px 30px rgba(0, 0, 0, 0.3);
}

.box.selected {
  transform: rotateY(180deg) translateZ(50px);
}

.box.revealed {
  transform: rotateY(180deg) translateZ(50px);
}

.box-front,
.box-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.box-front {
  background: linear-gradient(145deg, #00008b, #0000cd);
  transform: translateZ(1px);
}

.box-back {
  visibility: hidden;
  transform: rotateY(180deg) translateZ(1px);
}

.box-front::before,
.box-back::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 8px;
  background: linear-gradient(45deg, rgba(255, 255, 255, 0.1), rgba(255, 255, 255, 0));
  pointer-events: none;
}

.box-front::after,
.box-back::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 8px;
  box-shadow: inset 0 0 20px rgba(0, 0, 0, 0.2);
  pointer-events: none;
}

.box-front span,
.box-back span {
  font-size: 24px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  transform: translateZ(20px);
}

.results {
  text-align: center;
  padding: 20px;
  color: white;
}

.results-list {
  max-width: 500px;
  margin: 20px auto;
  background: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
}

.result-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  margin: 5px 0;
  background: rgb(30, 255, 0);
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.rank {
  font-weight: bold;
  color: #2385e7;
  width: 40px;
}

.player-name {
  flex: 1;
  text-align: left;
  margin: 0 20px;
  font-weight: bold;
  color: rgb(0, 0, 0);
}

.box-value {
  color: #42b983;
  font-weight: bold;
}

.reset-btn {
  background-color: rgb(0, 60, 255);
  margin-top: 20px;
  width: 100px;
  height: 70px;
  font-size: 15px;
  color: white;

}

.reset-btn:hover {
  background-color: #c0392b;
}

.value-list {
  background-color: rgba(0, 0, 0, 0.5);

  /*background: rgba(201, 10, 10, 0.7);*/
  border-radius: 12px;
  padding: 15px;
  color: rgb(255, 255, 255);
}

.value-item {
  padding: 8px 12px;
  margin: 5px 0;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  font-weight: bold;
  transition: all 0.3s ease;
}

.value-item.revealed {
  color: #ff4444;
  text-decoration: line-through;
  opacity: 0.7;
}

.value-item:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateX(5px);
}

.orientation-warning {
  position: fixed;
  z-index: 9999;
  inset: 0;
  background: #111c;
  display: flex;
  align-items: center;
  justify-content: center;
}
.orientation-message {
  background: #fff;
  color: #c00;
  padding: 32px 24px;
  border-radius: 18px;
  font-size: 1.3rem;
  text-align: center;
  box-shadow: 0 4px 32px #0002;
}
</style>