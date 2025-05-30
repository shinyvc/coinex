<script setup lang="ts">
import MarketView from '@/views/trade/main/components/market.vue'
import ChartView from '@/views/trade/main/components/chart.vue'
import TradeView from '@/views/trade/main/components/trade.vue'
import OrderbookView from '@/views/trade/main/components/orderbook.vue'
import FilledOrderView from '@/views/trade/main/components/filledOrder.vue'
import { ref, onMounted, onUnmounted, computed, watch} from 'vue'
import { useOperationStore } from "@/stores/operationStore";

const operationStore = useOperationStore()

// 响应式布局控制
const isMobile = ref(window.innerWidth < 768)
const isTablet = ref(window.innerWidth >= 768 && window.innerWidth < 1200)
const isDesktop = ref(window.innerWidth >= 1200)

// 获取当前交易对信息
const tickerColor = ref('#FFFFF0');
const tickerClose = ref('0');
const tickerOpen = ref('0');
const tickerSymbol = ref('');

const handleResize = () => {
  isMobile.value = window.innerWidth < 768
  isTablet.value = window.innerWidth >= 768 && window.innerWidth < 1200
  isDesktop.value = window.innerWidth >= 1200
}

onMounted(() => {
  window.addEventListener('resize', handleResize)

  // 初始化价格相关数据
  tickerColor.value = operationStore.getTicker.color
  tickerClose.value = operationStore.getTicker.close
  tickerOpen.value = operationStore.getTicker.open
  tickerSymbol.value = operationStore.getTicker.symbol
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})

// 监听价格变化
watch(
    operationStore.getTicker,
    (newTicker, oldTicker) => {
    tickerColor.value = newTicker.color
    tickerClose.value = newTicker.close
    tickerOpen.value = newTicker.open
    tickerSymbol.value = newTicker.symbol
    }
);
</script>

<template>
  <div class="trade-layout" :class="{'mobile-layout': isMobile, 'tablet-layout': isTablet, 'desktop-layout': isDesktop}">
    <!-- 左侧市场面板 - 仅在平板和桌面显示 -->
    <div v-if="!isMobile" class="trade-panel market-panel">
      <MarketView/>
    </div>
    
    <!-- 中间内容区域 -->
    <div class="trade-panel center-panel">
      <!-- 仅在移动端显示的交易对信息条 -->
      <div v-if="isMobile" class="symbol-header">
        <div class="symbol-info">
          <span class="symbol-name">{{ tickerSymbol }}</span>
          <span class="price" :style="{ color: tickerColor }">{{ tickerClose }}</span>
        </div>
      </div>
      
      <!-- K线图表区域 -->
      <div class="chart-container">
        <ChartView/>
      </div>
      
      <!-- 交易面板 -->
      <div class="trade-container">
        <TradeView/>
      </div>
    </div>
    
    <!-- 订单簿面板 - 仅在桌面显示 -->
    <div v-if="isDesktop" class="trade-panel orderbook-panel">
      <OrderbookView/>
    </div>
    
    <!-- 最新成交面板 - 仅在桌面显示 -->
    <div v-if="isDesktop" class="trade-panel filledorders-panel">
      <div class="filled-orders-header">
        <h3 class="panel-title">最新成交</h3>
      </div>
      <FilledOrderView/>
    </div>
    
    <!-- 移动端和平板的标签页视图 -->
    <div v-if="!isDesktop" class="mobile-tabs trade-panel">
      <el-tabs type="card">
        <el-tab-pane label="订单簿">
          <div class="tab-content">
            <OrderbookView/>
          </div>
        </el-tab-pane>
        <el-tab-pane label="成交">
          <div class="tab-content">
            <FilledOrderView/>
          </div>
        </el-tab-pane>
        <el-tab-pane v-if="isMobile" label="市场">
          <div class="tab-content">
            <MarketView/>
          </div>
        </el-tab-pane>
      </el-tabs>
    </div>
  </div>
</template>

<style scoped lang="scss">
.trade-layout {
  width: 100%;
  height: 100%;
  display: grid;
  gap: 3px;
  background-color: rgba(0, 0, 0, 0.15);
  
  &.mobile-layout {
    grid-template-columns: 1fr;
    grid-template-rows: auto auto auto;
    height: auto;
  }
  
  &.tablet-layout {
    grid-template-columns: 240px 1fr;
    grid-template-rows: 1fr auto;
    grid-template-areas: 
      "market chart"
      "market trade"
      "market tabs";
    height: calc(100vh - 64px - 200px);
  }
  
  &.desktop-layout {
    grid-template-columns: 280px minmax(0, 3fr) minmax(0, 0.75fr) minmax(0, 0.9fr);
    grid-template-rows: minmax(0, 1fr);
    height: calc(100vh - 64px - 200px);
  }
}

.trade-panel {
  background-color: var(--binance-bg-base);
  border-radius: 1px;
  border: none;
  overflow: hidden;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.1);
  
  &.market-panel {
    height: 100%;
    max-height: 100%;
    overflow-y: auto;
  }
  
  &.center-panel {
    display: flex;
    flex-direction: column;
    gap: 3px;
    padding: 0;
    background-color: rgba(0, 0, 0, 0.12);
    height: 100%;
  }
  
  &.orderbook-panel, &.filledorders-panel {
    height: 100%;
    display: flex;
    flex-direction: column;
    overflow-y: hidden;
  }
}

.symbol-header {
  display: flex;
  align-items: center;
  padding: var(--binance-spacing-sm) var(--binance-spacing-md);
  border-radius: 2px 2px 0 0;
  background-color: var(--binance-bg-base);
  border-bottom: 1px solid rgba(255, 255, 255, 0.07);
  
  .symbol-info {
    display: flex;
    align-items: center;
    gap: var(--binance-spacing-md);
    
    .symbol-name {
      font-weight: 600;
      font-size: 16px;
    }
    
    .price {
      font-weight: 600;
      font-size: 16px;
    }
  }
}

.chart-container {
  flex: 2.5;
  min-height: 400px;
  border-radius: 3px;
  overflow: hidden;
  background-color: #1e2026;
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.2);
  
  @media (max-width: 768px) {
    min-height: 400px;
  }
}

.trade-container {
  flex: 1;
  margin-top: 0;
  border-radius: 3px;
  background-color: var(--binance-bg-base);
  border: none;
  overflow: auto;
  min-height: 280px;
  
  @media (max-width: 768px) {
    padding: 0;
    height: auto;
    min-height: 450px;
  }
}

.filled-orders-header {
  padding: 8px 10px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.07);
  flex-shrink: 0;
  background-color: rgba(0, 0, 0, 0.05);
}

.panel-title {
  font-size: 13px;
  font-weight: 500;
  margin: 0;
  color: var(--binance-text-primary);
}

.mobile-tabs {
  margin-top: 0;
  border-radius: 3px;
  background-color: var(--binance-bg-base);
  border: none;
  overflow: hidden;
  
  .tab-content {
    height: 300px;
    overflow: auto;
  }
  
  :deep(.el-tabs__header) {
    margin: 0;
    background-color: var(--binance-bg-secondary);
    border-bottom: 1px solid rgba(255, 255, 255, 0.07);
  }
  
  :deep(.el-tabs__nav) {
    border: none !important;
  }
  
  :deep(.el-tabs__item) {
    border: none !important;
    height: 40px;
    color: var(--binance-text-secondary);
    
    &.is-active {
      color: var(--binance-primary);
      background-color: rgba(240, 185, 11, 0.05);
      font-weight: 500;
    }
    
    &:hover {
      color: var(--binance-text-primary);
      background-color: rgba(255, 255, 255, 0.03);
    }
  }
  
  :deep(.el-tabs__active-bar) {
    background-color: var(--binance-primary);
    height: 3px;
  }
}

@media (max-width: 1200px) {
  .tablet-layout .market-panel {
    grid-area: market;
  }
  
  .tablet-layout .center-panel {
    grid-area: chart;
  }
  
  .tablet-layout .trade-container {
    grid-area: trade;
    min-height: 360px;
  }
  
  .tablet-layout .mobile-tabs {
    grid-area: tabs;
  }
}
</style>