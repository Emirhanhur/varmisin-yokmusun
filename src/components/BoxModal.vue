<template>
  <div class="modal-overlay">
    <div class="modal-content">
      <img v-if="!opened" src="./box_close.png" class="box-img" alt="Kapalı Kutu" />
      <div v-else class="open-box-content bg-primary">
        <img src="./box_open.png" class="box-img" alt="Açık Kutu" />
        <div class="amount">{{ amount }} TL</div>
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
  }
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  left: 0; top: 0; width: 100vw; height: 100vh;
  background: rgba(255, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}
.modal-content {
  background: #222;
  border-radius: 16px;
  padding: 32px 48px;
  text-align: center;
  position: relative;
  min-width: 320px;
}
.box-img {
  width: 220px;
  max-width: 80vw;
  margin-bottom: 24px;
  background-color:blue;
}
.amount {
  font-size: 2.5rem;
  color: #fff;
  margin: 24px 0;
  font-weight: bold;
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
</style> 