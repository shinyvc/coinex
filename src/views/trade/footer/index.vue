<script setup lang="ts">
import CurrentOrderView from "@/views/trade/footer/components/currentOrder.vue"
import HistoricalOrderView from "@/views/trade/footer/components/historicalOrder.vue"
import type {TabsPaneContext} from 'element-plus'
import {ref} from "vue";

var curTab: string
const currentRef = ref(null);
const historyRef = ref(null);
const activeName = ref('current')

const onTabsClick = (tab: TabsPaneContext, _: Event) => {
  if (curTab === tab.props.name) {
    return
  }
  curTab = tab.props.name as string
  switch (curTab) {
    case "current":
      currentRef.value.loadOrderList();
      break
    case "history":
      historyRef.value.loadOrderList();
      break
  }
}

</script>

<template>
  <div class="footer-container">
    <el-tabs v-model="activeName" @tab-click="onTabsClick" class="order-tabs">
      <el-tab-pane class="current-order-view" label="当前委托" name="current">
        <CurrentOrderView ref="currentRef" :history="false"/>
      </el-tab-pane>
      <el-tab-pane class="historical-order-view" label="历史委托" name="history">
        <HistoricalOrderView ref="historyRef" :history="true"/>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<style scoped lang="scss">
.footer-container {
  width: 100%;
  min-height: 200px;
  padding: 0;
  
  @media (max-width: 768px) {
    padding: 0;
  }
}

.order-tabs {
  width: 100%;
  
  :deep(.el-tabs__header) {
    padding: 0 var(--binance-spacing-md);
    margin-bottom: 8px;
    border-bottom: 1px solid var(--binance-border-base);
    
    @media (max-width: 768px) {
      padding: 0 var(--binance-spacing-sm);
      margin-bottom: 6px;
    }
  }
  
  :deep(.el-tabs__nav-wrap::after) {
    background-color: var(--binance-border-base);
  }
  
  :deep(.el-tabs__item) {
    height: 40px;
    line-height: 40px;
    font-size: 14px;
    font-weight: 500;
    
    &.is-active {
      color: var(--binance-primary) !important;
    }
  }
  
  :deep(.el-tabs__active-bar) {
    height: 2px;
    background-color: var(--binance-primary) !important;
  }
  
  :deep(.el-tabs__content) {
    width: 100%;
    height: calc(100% - 48px); /* 减去标签头的高度 */
    padding: 0 var(--binance-spacing-md);
    
    @media (max-width: 768px) {
      padding: 0 var(--binance-spacing-sm);
    }
  }
  
  :deep(.el-tab-pane) {
    width: 100%;
    height: 100%;
    padding: 0;
  }
}

.current-order-view, .historical-order-view {
  width: 100%;
}
</style>
