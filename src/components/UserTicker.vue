<template>
  <div class="glass-card ticker-card">
    <div class="ticker-wrapper">
      <div v-for="(item, index) in tickerItems" :key="index" class="ticker-item">
        <span>{{ item.text }}</span>
        <span style="color: var(--success-color); font-weight: bold;">[生成成功]</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tickerItems = ref([
  { text: '用户 138****8892 成功生成了 [关于进一步加强安全生产检查的通知] 标准公文请示' },
  { text: '用户 159****3104 成功合成了 [某市直机关2025年度党风廉政建设总结] 述职报告' },
  { text: '用户 186****6617 成功解析了 [主要领导在深化作风建设动员会上的讲话稿] 会演发言' },
  { text: '用户 135****9023 成功策划了 [优化营商环境专项督查整改措施] 专项排查报告' },
  { text: '用户 173****5180 成功生成了 [基层社区数字治理提质增效工作总结] 汇报材料' },
]);
</script>

<style scoped>
.ticker-card {
  padding: 0.6rem 1rem;
  margin-bottom: 1.25rem;
  overflow: hidden;
}

.ticker-wrapper {
  display: flex;
  gap: 2rem;
  white-space: nowrap;
  animation: ticker 25s linear infinite;
}

.ticker-wrapper:hover {
  animation-play-state: paused;
}

.ticker-item {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.8rem;
  color: var(--text-secondary);
}

@keyframes ticker {
  0% {
    transform: translateX(100%);
  }
  100% {
    transform: translateX(-100%);
  }
}
</style>
