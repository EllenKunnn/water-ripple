<script setup>
import { onMounted, onBeforeUnmount, reactive, ref } from 'vue'

const canvas = ref(null)
let ctx = null


const points = ref([])
const config = reactive({
    size: 64,
    radius: window.innerWidth * 0.1,
    width: window.innerWidth, 
    height: window.innerHeight,
    maxAge: 64
})

onMounted(() => {
    initTexture()
    init()


})

const init = () => {
    canvas.value.addEventListener('mousemove', handleMouseMove);
    tick()
}

const tick = () => {
    animationFrameId = requestAnimationFrame(update);

}

const initTexture = () => {
    ctx = canvas.value.getContext('2d')
    clear()
}
const clear = (() => {
    ctx.fillStyle = 'black'
    ctx.fillRect(0, 0, config.width, config.height);
})

const handleMouseMove = (event) => {
  const rect = canvas.value.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  addPoint({x, y})
//   console.log(`Mouse position: x=${x}, y=${y}`);
};



let animationFrameId = null;
const update = () => {
    console.log(update)
  if (ctx) {
    clear()
    points.value.forEach((point, index) => {
        console.log('point===>',point, index)
        point.age += 1;
        if(point.age > config.maxAge) {
            points.splice(index, 1);
        }
    })
    points.value.forEach(point => {
        drawPoints(point);
    })
    // 更新水纹纹理的逻辑
    // 例如，绘制一些动画效果

    // 这里可以添加更多绘制逻辑
    // points
  }
};


const addPoint = (point) => {
    points.value.push({ x: point.x, y:point.y, age: 0 })
}


const drawPoints = (point) => {
    console.log('=======>',point)
    // Convert normalized position into canvas coordinates
    let pos = {
        x: point.x * config.width,
        y: point.y * config.height
    }
    const radius = config.radius;
    
    
    ctx.beginPath();
    ctx.arc(pos.x, pos.y, radius, 0, Math.PI * 2);
    ctx.fill();
}

onBeforeUnmount(() => {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId);
  }
});


</script>
<template>
    <div>
        <canvas ref="canvas" id="canvas"></canvas>
    </div>
</template>
