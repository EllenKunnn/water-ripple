<template>
  <div>
    <WaterTexture ref="waterTexture" :debug="true" />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import WaterTexture from './components/WaterTexture.vue';


// 1. 监听鼠标移动捕获为数组points[]中的一个新波纹
onMounted(() => {
  window.addEventListener('mousemove', onMouseMove);
  tick();
});

onBeforeUnmount(() => {
  window.removeEventListener('mousemove', onMouseMove);
});

const waterTexture = ref(null);

const onMouseMove = (event) => {
  const point = {
    x: event.clientX / window.innerWidth,
    y: event.clientY / window.innerHeight,
  };
  if (waterTexture.value) {
    waterTexture.value.addPoint(point);
  }
};

// 
const tick = () => {
  if (waterTexture.value) {
    waterTexture.value.update();
  }
  requestAnimationFrame(tick);
};


</script>