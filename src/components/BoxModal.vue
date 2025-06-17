<template>
  <div class="modal-overlay">
    <div class="modal-content">
      <img v-if="!opened" src="./box_close.png" class="box-img" alt="Kapalı Kutu" />
      <div v-else class="open-box-content bg-primary">
        <img src="./box_open.png" class="box-img" alt="Açık Kutu" />
        <div class="amount">{{ formatNumber(amount) }} TL</div>
      </div>
      <button v-if="opened" @click="$emit('close')">Kapat</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    opened: Boolean,
    amount: Number
  },
  methods: {
    formatNumber(number) {
      if (number >= 1000) {
        return number.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
      }
      return number;
    }
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  left: 0; top: 0; width: 100vw; height: 100vh;
  background: rgba(216, 216, 216, 0.7);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  opacity: 0;
  animation: fadeIn 0.3s ease forwards;
}
.modal-content {
  background: #222;
  border-radius: 16px;
  padding: 32px 48px;
  text-align: center;
  position: relative;
  min-width: 320px;
  transform: scale(0.8);
  animation: scaleIn 0.3s ease forwards;
}
.box-img {
  width: 220px;
  max-width: 80vw;
  margin-bottom: 24px;
  transition: all 0.3s ease;
}
.amount {
  font-size: 2.5rem;
  color: #fff;
  margin: 24px 0;
  font-weight: bold;
  opacity: 0;
  animation: fadeIn 0.3s ease forwards 0.3s;
}
button {
  padding: 10px 24px;
  font-size: 1.2rem;
  border-radius: 8px;
  border: none;
  background:rgb(0, 0, 0);
  color: #fff;
  cursor: pointer;
  margin-top: 16px;
}
.open-box-content {
  position: relative;
}
.open-box-content .box-img {
  animation: boxOpen 0.5s ease forwards;
}
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
@keyframes scaleIn {
  from {
    transform: scale(0.8);
  }
  to {
    transform: scale(1);
  }
}
@keyframes boxOpen {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.5;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}
</style> 