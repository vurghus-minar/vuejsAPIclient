<template>
  <transition name="fade">
    <div class="overlay"></div>
  </transition>
</template>
<style scoped>
.overlay {
  height: 100%;
  width: 100%;
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  left: 0;
  top: 0;
  background-color: rgb(0, 0, 0); /* Black fallback color */
  background-color: rgba(0, 0, 0, 0.9); /* Black w/opacity */
  overflow-x: hidden; /* Disable horizontal scroll */
}
</style>

