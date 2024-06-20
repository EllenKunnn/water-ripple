<template>
    <canvas ref="canvas"></canvas>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const canvas = ref(null);
// 波纹数组, 捕获鼠标移动事件，向内新增
const points = ref([]);
const size = 64;
const maxAge = 64;
let width = window.innerWidth;
let height = window.innerHeight;
// 波纹半径
let radius = width * 0.05;
let ctx = null;


const initTexture = () => {
    canvas.value.width = width;
    canvas.value.height = height;
    ctx = canvas.value.getContext('2d');
    clear();
    document.body.append(canvas.value);
};

const clear = () => {
    ctx.fillStyle = 'black';
    ctx.fillRect(0, 0, window.innerWidth, window.innerHeight);
};

const last = ref(null);

const addPoint = (point) => {
    let force = 0;
    let vx = 0;
    let vy = 0;
    if(last.value){
        const relativeX = point.x - last.value.x;
        const relativeY = point.y - last.value.y;

        const distanceSquared = relativeX * relativeX + relativeY * relativeY;
        
        const distance = Math.sqrt(distanceSquared);

        vx = relativeX / distance;
        vy = relativeY / distance;
        
        force = Math.min(distanceSquared * 10000,1.);
    }
    
    last.value = {
        x: point.x,
        y: point.y
    }
    points.value.push({ x: point.x, y: point.y, age: 0, force, vx, vy });


    // points.value.push({ x: point.x, y: point.y, age: 0 });
};

// 清空画布，将超出生命周期的波纹回收，绘制波纹
// const update = () => {
//     clear();
//     points.value.forEach((point, i) => {
//         point.age += 1;
//         if (point.age > maxAge) {
//             points.value.splice(i, 1);
//         }
//     });
//     points.value.forEach((point) => {
//         drawPoint(point);
//     });
// };



// 带矢量的update
const update = () => {
    clear();
    let agePart = 1.0 / maxAge;
    points.value.forEach((point, i) => {
        
        let slowAsOlder = (1.0- point.age / maxAge)

        let force = point.force * agePart * slowAsOlder;
    
        point.x += point.vx * force;
        point.y += point.vy * force;
        point.age += 1;
        
        if (point.age > maxAge) {
            points.value.splice(i, 1);
        }
    });
    points.value.forEach((point) => {
        drawPoint(point);
    });
};

const easeOutSine = (t, b, c, d) => {
  return c * Math.sin((t / d) * (Math.PI / 2)) + b;
};

const easeOutQuad = (t, b, c, d) => {
  t /= d;
  return -c * t * (t - 2) + b;
};



// 绘制节点
const drawPoint = (point) => {
    
    // 因为前面做了归一化处理，这里需要还原
    const pos = {
        x: point.x * width,
        y: point.y * height,
    };


    // 4 使用缓动函数
    // 不是简单地让效果逐渐减弱直至消失，而是让效果在开始时先增强，然后再减弱。
    let intensity = 1.;
    if (point.age < maxAge * 0.3) {
        intensity = easeOutSine(point.age / (maxAge * 0.3), 0, 1, 1);
    } else {
        intensity = easeOutQuad(
        1 - (point.age - maxAge * 0.3) / (maxAge * 0.7),
        0,
        1,
        1
        );
    }
    intensity *= point.force;
    
    // let intensity = 1 - point.age / maxAge;

    const color = '255,255,255';
    const offset = width * 5;

    ctx.shadowOffsetX = offset;
    ctx.shadowOffsetY = offset;
    ctx.shadowBlur = radius * 1;
    // 阴影的渐变效果
    ctx.shadowColor = `rgba(${color},${0.2 * intensity})`;

    ctx.beginPath();
    ctx.fillStyle = 'rgba(255,0,0,1)';
    ctx.arc(pos.x - offset, pos.y - offset, radius, 0, Math.PI * 2);
    // ctx.arc(pos.x , pos.y, radius, 0, Math.PI * 2);
    ctx.fill();
 
};

onMounted(() => {
    initTexture();
    const tick = () => {
        update();
        requestAnimationFrame(tick);
    };
    tick();
});

defineExpose({addPoint})
</script>