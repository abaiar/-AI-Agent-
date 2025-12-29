<template>
  <div class="map-container">
    
    <svg 
      viewBox="0 0 1000 800" 
      preserveAspectRatio="xMidYMid meet"
      class="responsive-svg"
    >
      
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
// 1. 引入 Vue 的 computed 用于计算属性
import { computed } from 'vue';
// 2. 引入模拟的地图数据（景点和边）
import { spots, edges } from '../data/mapData.js';

// 3. 定义 Props：接收父组件传来的当前路径数组
const props = defineProps({
  activePath: {
    type: Array, // 类型必须是数组
    default: () => [] // 默认值为空数组
  }
});

// 4. 定义 Emits：声明组件会向外触发 'spot-click' 事件
defineEmits(['spot-click']);

// 5. 辅助函数：防止因 ID 不存在导致坐标读取报错（防御性编程）
const getSpot = (id) => {
  return spots[id] || { x: 0, y: 0 }; // 如果找不到 ID，返回 0,0 坐标
};

// 6. 核心逻辑：判断某个景点是否在当前导航路径中
const isSpotActive = (id) => {
  // 检查 activePath 是否存在且包含该 ID
  return props.activePath && props.activePath.includes(id);
};

// 7. 计算属性：将路径数组转换为 SVG polyline 需要的字符串格式
// 例如：['S01', 'S02'] -> "100,100 200,200"
const pathPointsString = computed(() => {
  // 如果路径点不足 2 个，无法连线，返回空
  if (!props.activePath || props.activePath.length < 2) return "";
  
  return props.activePath
    .map(id => {
      const spot = spots[id];
      // 确保点存在，格式化为 "x,y"
      return spot ? `${spot.x},${spot.y}` : '';
    })
    .filter(p => p !== '') // 过滤掉无效数据
    .join(" "); // 用空格连接所有坐标
});

// 8. 简单的计算属性别名，用于模板中 v-if 的长度判断
const pathPoints = computed(() => props.activePath);
</script>

<style scoped>
/* 容器样式：圆角、阴影、背景色 */
.map-container {
  width: 100%;
  height: 100%;
  background: #f9f9f9;
  border-radius: 12px;
  box-shadow: inset 0 0 20px rgba(0,0,0,0.05); /* 内阴影增加质感 */
  overflow: hidden; /* 防止 SVG 超出圆角 */
  display: flex;
  justify-content: center;
  align-items: center;
}

/* SVG 自适应：确保地图在容器内完整显示 */
.responsive-svg {
  width: 100%;
  height: 100%;
  max-width: 100%;
  max-height: 100%;
  display: block; /* 消除行内元素默认间隙 */
}

/* 圆点基础样式：蓝色背景，白色描边 */
.spot-circle {
  fill: #4a90e2;
  stroke: white;
  stroke-width: 2px;
  transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1); /* 贝塞尔曲线过渡 */
}

/* 交互：鼠标悬停在组上时，圆点变大变橙 */
.spot-group:hover .spot-circle {
  r: 12;
  fill: #ff9800;
  cursor: pointer;
}

/* 状态：激活（导航中）的圆点，红色、金色描边 */
.spot-circle.active {
  fill: #ff5252 !important; /* 强制覆盖 */
  r: 12;
  stroke: #ffd700;
  stroke-width: 3px;
}

/* 文字基础样式：白色、无鼠标事件（防遮挡） */
.spot-label {
  font-size: 12px;
  fill: white;
  font-family: sans-serif;
  pointer-events: none; /* 关键：让点击穿透文字打到圆点上 */
  transition: all 0.3s ease;
  text-shadow: 0 1px 2px white;
}

/* 交互：悬停时文字变黑加粗 */
.spot-group:hover .spot-label {
  fill: #333;
  font-weight: bold;
}

/* 状态：激活时文字变大变红 */
.spot-label.active {
  fill: #ff5252 !important;
  font-weight: bold;
  font-size: 14px;
}

/* 背景图样式 */
.background-image {
  opacity: 1;
}

/* 动画：路径描边动画定义 */
.path-animation {
  stroke-dasharray: 5000; /* 设置超长虚线间隔 */
  stroke-dashoffset: 5000; /* 初始偏移量，隐藏线条 */
  animation: dash 2.5s ease-out forwards; /* 播放动画 */
}

/* 关键帧：将偏移量归零，实现线条生长效果 */
@keyframes dash {
  to {
    stroke-dashoffset: 0;
  }
}
</style>
