<template>
  <div class="bug-stat-container">
    <!-- 卡片容器 -->
    <el-card class="stat-card">
      <!-- 头部标题区域 -->
      <div class="card-header">
        <h2 class="card-title">近7天BUG提交统计</h2>
        <div class="total-count">
          累计提交：<span class="count-value">{{ data.total }}</span> 个BUG
        </div>
      </div>

      <!-- 图表区域 -->
      <div class="chart-wrapper">
        <div id="line" class="chart-container"></div>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import * as echarts from 'echarts';
import { onMounted, reactive, watchEffect } from "vue";
import axios from "@/utils/axios.js";
import { ElMessage, ElLoading } from "element-plus";

const data = reactive({
  total: 0,
  isLoading: false
});

const lineOption = {
  // 图表标题样式优化
  title: {
    text: '每日提交趋势',
    left: 'center',
    textStyle: {
      fontSize: 16,
      color: '#333'
    }
  },
  // 提示框样式优化
  tooltip: {
    trigger: 'axis',
    backgroundColor: 'rgba(255, 255, 255, 0.9)',
    borderColor: '#eee',
    borderWidth: 1,
    textStyle: { color: '#333' },
    formatter: '{b}: {c} 个'
  },
  legend: {
    data: ['提交量'],
    bottom: 10
  },
  grid: {
    left: '3%',
    right: '4%',
    bottom: '15%',
    top: '15%',
    containLabel: true
  },
  xAxis: {
    type: 'category',
    boundaryGap: false,
    data: [],
    axisLine: { lineStyle: { color: '#ddd' } },
    axisLabel: {
      color: '#666',
      interval: 0
    }
  },
  yAxis: {
    type: 'value',
    min: 0,
    interval: 1,
    axisLine: { show: false },
    axisLabel: { color: '#666' },
    splitLine: { lineStyle: { color: '#f5f5f5' } }
  },
  series: [
    {
      name: '提交量',
      type: 'line',
      smooth: true,
      data: [],
      symbol: 'circle',
      symbolSize: 6,
      lineStyle: { width: 2, color: '#409eff' },
      itemStyle: { color: '#409eff' },
      areaStyle: {
        color: {
          type: 'linear',
          x: 0,
          y: 0,
          x2: 0,
          y2: 1,
          colorStops: [{
            offset: 0, color: 'rgba(64, 158, 255, 0.3)'
          }, {
            offset: 1, color: 'rgba(64, 158, 255, 0)'
          }]
        }
      }
    }
  ]
};

// 初始化图表
let lineChart = null;

onMounted(() => {
  // 初始化图表实例
  const chartDom = document.getElementById('line');
  lineChart = echarts.init(chartDom);

  // 加载数据
  fetchData();

  // 监听窗口大小变化，自适应图表
  window.addEventListener('resize', handleResize);

  // 组件卸载时清理
  return () => {
    window.removeEventListener('resize', handleResize);
    if (lineChart) {
      lineChart.dispose();
    }
  };
});

// 处理窗口大小变化
const handleResize = () => {
  if (lineChart) {
    lineChart.resize();
  }
};

// 统一获取数据方法
const fetchData = async () => {
  data.isLoading = true;
  const loading = ElLoading.service({
    target: '#line',
    text: '加载中...'
  });

  try {
    // 并行请求两个接口，提高效率
    const [lineRes, totalRes] = await Promise.all([
      axios.get("/bugs/lineData"),
      axios.get("/bugs/bugAll")
    ]);

    // 更新图表数据
    lineOption.xAxis.data = lineRes.data.date || [];
    lineOption.series[0].data = lineRes.data.count || [];
    lineChart.setOption(lineOption);

    // 更新总数
    data.total = totalRes.code === 0 ? (totalRes.data?.length || 0) : 0;

  } catch (error) {
    console.error("数据获取失败：", error);
    ElMessage.error("数据加载失败，请重试");
  } finally {
    data.isLoading = false;
    loading.close();
  }
};
</script>

<style scoped>
.bug-stat-container {
  padding: 20px;
  min-width: 600px;
}

.stat-card {
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  overflow: hidden;
}

.card-header {
  padding: 16px 20px;
  border-bottom: 1px solid #f5f5f5;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-title {
  margin: 0;
  font-size: 18px;
  font-weight: 500;
  color: #333;
}

.total-count {
  font-size: 14px;
  color: #666;
}

.count-value {
  color: #409eff;
  font-size: 16px;
  font-weight: 600;
  margin: 0 4px;
}

.chart-wrapper {
  padding: 20px;
}

.chart-container {
  width: 100%;
  height: 400px;
  border-radius: 8px;
  transition: all 0.3s ease;
}

/* 响应式调整 */
@media (max-width: 768px) {
  .bug-stat-container {
    padding: 10px;
    min-width: auto;
  }

  .chart-container {
    height: 300px;
  }

  .card-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
}
</style>