<template>
  <div :style="floatingStyle" class="floating-text" @click="onClick">
    <slot></slot>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted } from 'vue';
import randomInt from 'lodash/random';


interface FloatingTextProps {
  duration?: number; // 持续时间 单位 s 默认为 5
  maxHeight: number; // 最大高度 单位 px 默认为 500
  maxWidth: number; // 最大宽度 单位 px 默认为 500
  x?: number; // 水平位置 单位 px 默认为 随机
  y?: number; // 垂直位置 单位 px
  xDirection?: number; // 水平方向 >1向右, -1< 向左, 0 是不换动
  color?: string; // 字体颜色 默认为 red
  onClick?: () => void; // 点击事件
}

const props = withDefaults(defineProps<FloatingTextProps>(), {
  duration: 5,
  maxHeight: 500,
  maxWidth: 500,
  x: 0,
  y: 100,
  color: 'red', // 默认字体颜色
  onClick: () => {
  }
});

// 默认位置 
const realX = computed(() => {
  if ( props.x === 0 || props.x > props.maxWidth ) {
    console.warn('X is 0 or greater then MaxWidth, set to random value:', realX);
     return randomInt(0, props.maxWidth/2);
  } else {
    return props.x;
  }
});

// 默认位置 
const realY = computed(() => {
  if ( props.y === 0 || props.y > props.maxHeight ) {
    console.warn('Y is 0 or greater then maxHeight, set to random value:', realY);
    return randomInt(0, props.maxHeight/2);
  } else {
    return props.y;
  }
});

const realXDirection = computed(() => {
  return props.xDirection;
});

// 参数验证
onMounted(() => {
  if (props.duration <= 0) {
    console.warn('Duration should be greater than 0');
  }
  if (props.maxHeight <= 0) {
    console.warn('MaxHeight should be greater than 0');
  }
  if (props.maxWidth <= 0) {
    console.warn('MaxWidth should be greater than 0');
  }
});

const floatingStyle = computed(() => ({
  '--duration': `${props.duration}s`,
  '--maxHeight': `${props.maxHeight}px`,
  '--maxWidth': `${props.maxWidth}px`,
  '--startX': `${realX.value}px`,
  '--startY': `${realY.value}px`,
  '--color': `${props.color}`,
  '--xDirection': `${realXDirection.value}`,
}));
</script>

<style lang="scss" scoped>
.floating-text {
  width: 150px;
  position: absolute;
  left: var(--startX);
  bottom: var(--startY);
  color: var(--color);
  //文本超出时候省略
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;

  // 动画效果
  filter: brightness(1) blur(2px);
  animation: float var(--duration) linear infinite;
}

@function height25() {
  @return calc(-1 * var(--maxHeight) / 4);
}

@function height50() {
  @return calc(-2 * var(--maxHeight) / 4);
}

@function height75() {
  @return calc(-3 * var(--maxHeight) / 4);
}

@function height100() {
  @return calc(-4 * var(--maxHeight) / 4);
}

@function width25() {
  // 1/10
  @return calc(var(--xDirection) * 1 * var(--maxWidth) / 10);
}

@function width50() {
  // 2/10
  @return calc(var(--xDirection) * 2 * var(--maxWidth) / 10);
}

@function width75() {
  // 回到 2/10 的宽度
  @return calc(var(--xDirection) * 1 * var(--maxWidth) / 10);
}

@function width100() {
  // 回到 原位置
  @return var(--startX);
}

@keyframes float {
  // teanslate3d(x, y, z) 3D移动 参数是绝对位置

  0% {
    transform: scale(0.5);  /* 起始位置, 字体较小 */
    opacity: 0.5;
    filter: brightness(0.5) blur(2px);
  }
  25% {
    transform: translate3d(width25(), height25(), 200px) scale(1);  /* 向上移动，字体增大 */
    opacity: 1;
    filter: brightness(0.8) blur(1px);
    }
  
    50% {
      transform: translate3d(width50(), height50(), 500px) scale(1.5); /* 中间位置，字体最大 */
      opacity: 1;
      filter: brightness(1.2) blur(0);
    }
  75% {
    transform: translate3d(width75(), height75(), 200px) scale(1);  /* 向上移动，字体减小 */
    opacity: 1;
    filter: brightness(0.8) blur(1px);
  }
  100% {
    transform: translate3d(0%, height100(), 100px) scale(0.5);  /* 终点位置，字体较小 */
    opacity: 0.5;
    filter: brightness(0.5) blur(2px);
  }
}
</style>
