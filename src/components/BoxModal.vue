<template>
  <div class="modal-overlay">
    <div class="modal-content">
      <img v-if="!opened" src="./box_close.png" class="box-img" alt="Kapalı Kutu" />
      <div v-else class="open-box-content">
        <img src="./box_open.png" class="box-img" alt="Açık Kutu" />
        <div class="amount-on-lid">{{ formatNumber(amount) }} TL</div>
      </div>
      <button class="w-25" v-if="opened" @click="$emit('close')">Kapat</button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    opened: Boolean,
    amount: Number
  },
  watch: {
    opened(newValue) {
      if (newValue) {
        setTimeout(() => {
          this.$emit('close')
        }, 10000)
      }
    }
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
* {
  margin: 0;
  padding: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal-overlay {
  position: fixed;
  left: 0;right: 0; top: 0; width: 100vw; height: 100vh;
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  opacity: 0;
  animation: fadeIn 0.3s ease forwards;
}
.modal-content {
  border-radius: 16px;
  /* padding: 100px 300px; */
  text-align: center;
  position: relative;
  /* min-width: 320px; */
  transform: scale(0.8);
  animation: scaleIn 0.3s ease forwards;
}
.box-img {
  width: 280px;
  max-width: 90w;
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
  display: flex;
  justify-content: center;
  align-items: center;
}
.open-box-content .box-img {
  animation: boxOpen 0.5s ease forwards;
}
.amount-on-lid {
  position: absolute;
  top: 28px;
  left: 0;
  width: 100%;
  text-align: center;
  font-size: 2rem;
  color: #fff;
  font-weight: bold;
  text-shadow: 2px 2px 8px #000;
  pointer-events: none;
  z-index: 2;
  rotate: 3deg;
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