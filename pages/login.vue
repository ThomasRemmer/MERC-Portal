<template>
  <div class="tv-bg">
    <div :class="['login-bg', { 'crt-on': crtOn }]">
      <LoginForm />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
const crtOn = ref(false);
onMounted(() => {
  crtOn.value = false;
  setTimeout(() => {
    crtOn.value = true;
  }, 50);
});
</script>

<style scoped>
.tv-bg {
  min-height: 100vh;
  min-width: 100vw;
  background: #000;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 0;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.login-bg {
  min-height: 480px;
  min-width: 420px;
  background: transparent;
  z-index: 2;
  width: 100vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transform: scaleY(0);
  opacity: 0;
  transition: transform 1.1s cubic-bezier(0.23, 1, 0.32, 1), opacity 0.6s;
  transform-origin: center;
}
.login-bg.crt-on {
  transform: scaleY(1);
  opacity: 1;
}
.tv-bg::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 1;
  background: repeating-linear-gradient(
    to bottom,
    rgba(57, 255, 20, 0.07) 0px,
    rgba(57, 255, 20, 0.07) 2px,
    transparent 2px,
    transparent 18px
  );
  -webkit-mask-image: radial-gradient(
    ellipse 60% 40% at 50% 55%,
    #000 70%,
    transparent 100%
  );
  mask-image: radial-gradient(
    ellipse 60% 40% at 50% 55%,
    #000 70%,
    transparent 100%
  );
  transform: scaleX(1.12) scaleY(1.18);
}
.tv-bg::after {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  height: 8px;
  top: -8px;
  z-index: 0;
  background: linear-gradient(
    90deg,
    transparent 0%,
    rgba(57, 255, 20, 0) 0%,
    rgba(57, 255, 20, 0.45) 20%,
    rgba(57, 255, 20, 0.85) 50%,
    rgba(57, 255, 20, 0.45) 80%,
    rgba(57, 255, 20, 0) 100%,
    transparent 100%
  );
  opacity: 0.55;
  filter: blur(1.5px);
  animation: scanline-move-down 10s linear infinite;
  animation-delay: 0s;
}
@keyframes scanline-move-down {
  0% {
    top: -8px;
    opacity: 0.22;
  }
  2% {
    opacity: 0.32;
  }
  5% {
    opacity: 0.22;
  }
  95% {
    opacity: 0.22;
  }
  98% {
    opacity: 0.32;
  }
  100% {
    top: calc(100% + 8px);
    opacity: 0.22;
  }
}
</style>
