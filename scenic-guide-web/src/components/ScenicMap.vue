<template>
  <div class="map-container">
    <svg 
      viewBox="0 0 1000 800" 
      preserveAspectRatio="xMidYMid meet"
      class="responsive-svg"
    >
          <!-- 背景图片图层 -->
      <image
        href="/map.png"
        x="0"
        y="0"
        width="1000"
        height="800"
        preserveAspectRatio="xMidYMid meet"
        class="background-image"
      />

      <line
        v-for="(edge, index) in edges"
        :key="`edge-${index}`"
        :x1="getSpot(edge[0]).x"
        :y1="getSpot(edge[0]).y"
        :x2="getSpot(edge[1]).x"
        :y2="getSpot(edge[1]).y"
        stroke="#e0e0e0"
        stroke-width="2"
      />

      <polyline
        v-if="pathPoints.length > 0"
        :key="pathPointsString"
        :points="pathPointsString"
        fill="none"
        stroke="#ff9800"
        stroke-width="6"
        stroke-linecap="round"
        stroke-linejoin="round"
        class="path-animation"
      />

      <g 
        v-for="(info, id) in spots" 
        :key="id"
        @click="$emit('spot-click', info.name)"
        class="spot-group"
      >
        <circle
          :cx="info.x"
          :cy="info.y"
          r="8"
          class="spot-circle"
          :class="{ 'active': isSpotActive(id) }"
        />
        
        <text
          :x="info.x"
          :y="info.y + 24"
          text-anchor="middle"
          class="spot-label"
          :class="{ 'active': isSpotActive(id) }"
        >
          {{ info.name }}
        </text>
      </g>
    </svg>
  </div>
</template>

<script setup>
import { computed } from 'vue';
import { spots, edges } from '../data/mapData.js';

const props = defineProps({
  activePath: {
    type: Array, // 期望格式 ['S01', 'S05', 'S06']
    default: () => []
  }
});

defineEmits(['spot-click']);

// 辅助函数：通过 ID 获取坐标对象
const getSpot = (id) => {
  return spots[id] || { x: 0, y: 0 }; // 防止 ID 不存在报错
};

// 核心修复：检查当前 ID 是否存在于 props.activePath 数组中
const isSpotActive = (id) => {
  return props.activePath && props.activePath.includes(id);
};

// 计算 SVG polyline 所需的 points 字符串
const pathPointsString = computed(() => {
  if (!props.activePath || props.activePath.length < 2) return "";
  
  return props.activePath
    .map(id => {
      const spot = spots[id];
      return spot ? `${spot.x},${spot.y}` : '';
    })
    .filter(p => p !== '') // 过滤掉无效点
    .join(" ");
});

// 暴露 pathPoints 给 template 检查长度
const pathPoints = computed(() => props.activePath);
</script>

<style scoped>
.map-container {
  width: 100%;
  height: 100%;
  background: #f9f9f9;
  border-radius: 12px;
  box-shadow: inset 0 0 20px rgba(0,0,0,0.05);
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
}

.responsive-svg {
  width: 100%;
  height: 100%;
  max-width: 100%;
  max-height: 100%;
  display: block;
}



/* 默认圆点样式 */
.spot-circle {
  fill: #4a90e2;
  stroke: white;
  stroke-width: 2px;
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1); /* 弹性过渡 */
}

/* 悬停样式：只改变半径和颜色，不改变位置 */
.spot-group:hover .spot-circle {
  r: 12;
  fill: #ff9800;
  cursor: pointer;
}

/* 激活圆点样式 */
.spot-circle.active {
  fill: #ff5252 !important;
  r: 12;
  stroke: #ffd700;
  stroke-width: 3px;
}

/* 默认文字样式 */
.spot-label {
  font-size: 12px;
  fill: white;
  font-family: sans-serif;
  pointer-events: none;
  transition: all 0.3s ease;
  text-shadow: 0 1px 2px white;
}

/* 悬停文字样式 */
.spot-group:hover .spot-label {
  fill: #333;
  font-weight: bold;
}

/* 激活文字样式 */
.spot-label.active {
  fill: #ff5252 !important;
  font-weight: bold;
  font-size: 14px;
}

/* 背景图片样式 */
.background-image {
  opacity: 1; /* 可以调整透明度 */
}

/* 路径动画 */
.path-animation {
  stroke-dasharray: 5000;
  stroke-dashoffset: 5000;
  animation: dash 2.5s ease-out forwards;
}

@keyframes dash {
  to {
    stroke-dashoffset: 0;
  }
}
</style>