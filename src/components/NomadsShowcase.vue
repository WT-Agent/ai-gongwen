<template>
  <section class="nomads-showcase-section">
    <div class="showcase-header">
      <div class="header-left">
        <h2 class="showcase-title">实战案例与标准公文模板库 (Nomads Showcase)</h2>
        <p class="showcase-subtitle">精选体制内标准公文与工作汇报场景，点击“一键套用”快速生成规范文案</p>
      </div>
      <span class="showcase-badge">已收录 {{ showcaseItems.length }} 个公文实战模板</span>
    </div>

    <div class="showcase-grid">
      <div 
        v-for="item in showcaseItems" 
        :key="item.id" 
        class="glass-card showcase-card"
      >
        <div class="card-header">
          <span class="scenario-tag">{{ item.tag }}</span>
          <span class="usage-count">{{ item.usageCount }} 次应用</span>
        </div>

        <div class="card-content">
          <h3 class="item-title">{{ item.title }}</h3>
          <p class="item-prompt">“{{ item.prompt }}”</p>
        </div>

        <div class="card-action">
          <button class="apply-btn" @click="applyTemplate(item)">
            <span>一键套用</span>
            <svg class="arrow-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <line x1="5" y1="12" x2="19" y2="12"></line>
              <polyline points="12 5 19 12 12 19"></polyline>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from 'vue';

export interface ShowcaseItem {
  id: string;
  tag: string;
  title: string;
  prompt: string;
  gongwenType?: string;
  orgType?: string;
  formatTier?: string;
  usageCount: string;
}

const emit = defineEmits<{
  (e: 'apply-template', payload: {
    prompt: string;
    gongwenType?: string;
    orgType?: string;
    formatTier?: string;
  }): void;
}>();

const showcaseItems = computed<ShowcaseItem[]>(() => [
  {
    id: 'gongwen-1',
    tag: '机关发文',
    title: '关于进一步加强安全生产隐患排查与整改的通知',
    prompt: '针对全市近期重点行业领域安全生产大检查，拟定一份发往各区县、各有关部门的标准公文通知，明确排查重点、整改时限与责任追究。',
    gongwenType: '党政机关标准公文请示与批复',
    orgType: '党委政府机关',
    formatTier: '严谨红头文件格式',
    usageCount: '41.2k'
  },
  {
    id: 'gongwen-2',
    tag: '述职总结',
    title: '单位2025年度抓党建与业务融合述职报告',
    prompt: '围绕政治建设、服务大局、廉洁自律及存在短板四个维度，总结本单位全年在推进数字政务与便民服务方面的成绩与未来打算。',
    gongwenType: '年度/季度工作总结与述职报告',
    orgType: '事业单位',
    formatTier: '标准工作汇报格式',
    usageCount: '38.6k'
  },
  {
    id: 'gongwen-3',
    tag: '领导讲话',
    title: '主要领导在深化作风建设专项行动上的动员讲话',
    prompt: '结合贯彻落实中央八项规定精神，面向全系统中层以上干部撰写动员讲话稿，强调提高政治站位、狠抓落实与长效常态。',
    gongwenType: '领导讲话稿与会议发言材料',
    orgType: '国有企业',
    formatTier: '会议纪要演讲格式',
    usageCount: '35.4k'
  },
  {
    id: 'gongwen-4',
    tag: '专项整改',
    title: '机关作风效能建设与问题整改自查报告',
    prompt: '针对上级督查反馈的办事流程较繁琐、跨部门协同效率不足等问题，剖析思想根源，提出4条针对性强且可落地的整改举措。',
    gongwenType: '专项检查排查与整改报告',
    orgType: '党委政府机关',
    formatTier: '严谨红头文件格式',
    usageCount: '29.9k'
  },
  {
    id: 'gongwen-5',
    tag: '基层治理',
    title: '基层社区提升网格化精细化治理水平汇报材料',
    prompt: '总结社区在党员联户、志愿服务、矛盾纠纷调解及智慧社区平台应用方面的创新举措与典型案例，形成专题汇报材料。',
    gongwenType: '年度/季度工作总结与述职报告',
    orgType: '基层社区',
    formatTier: '标准工作汇报格式',
    usageCount: '27.8k'
  },
  {
    id: 'gongwen-6',
    tag: '请示批复',
    title: '关于请求批准实施机关办公自动化升级的请示',
    prompt: '向主管领导及上级主管部门请示关于采购升级无纸化办公系统与数据安全防护设备的必要性、资金预算与预期成效。',
    gongwenType: '党政机关标准公文请示与批复',
    orgType: '事业单位',
    formatTier: '严谨红头文件格式',
    usageCount: '24.5k'
  }
]);

function applyTemplate(item: ShowcaseItem) {
  emit('apply-template', {
    prompt: item.prompt,
    gongwenType: item.gongwenType,
    orgType: item.orgType,
    formatTier: item.formatTier
  });
}
</script>

<style scoped>
.nomads-showcase-section {
  margin-top: 2rem;
  width: 100%;
}

.showcase-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 1.25rem;
  padding-bottom: 0.75rem;
  border-bottom: 1px solid var(--card-border);
}

.showcase-title {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text-primary);
  background: var(--primary-gradient);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.showcase-subtitle {
  font-size: 0.825rem;
  color: var(--text-secondary);
  margin-top: 0.25rem;
}

.showcase-badge {
  font-size: 0.75rem;
  color: #a5b4fc;
  background: rgba(99, 102, 241, 0.12);
  border: 1px solid rgba(99, 102, 241, 0.25);
  padding: 4px 10px;
  border-radius: 20px;
}

.showcase-grid {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  gap: 1.25rem;
}

@media (min-width: 640px) {
  .showcase-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (min-width: 1024px) {
  .showcase-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

.showcase-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%;
  padding: 1.25rem;
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid var(--card-border);
  border-radius: 14px;
  transition: all 0.25s ease;
}

.showcase-card:hover {
  background: rgba(255, 255, 255, 0.05);
  border-color: rgba(99, 102, 241, 0.4);
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.scenario-tag {
  font-size: 0.75rem;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(168, 85, 247, 0.15);
  color: #c084fc;
  border: 1px solid rgba(168, 85, 247, 0.3);
}

.usage-count {
  font-size: 0.75rem;
  color: var(--text-secondary);
}

.card-content {
  margin-bottom: 1rem;
  flex: 1;
}

.item-title {
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: 0.4rem;
}

.item-prompt {
  font-size: 0.825rem;
  color: var(--text-secondary);
  line-height: 1.45;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  font-style: italic;
}

.card-action {
  padding-top: 0.75rem;
  border-top: 1px solid rgba(255, 255, 255, 0.04);
}

.apply-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 0.5rem 1rem;
  background: rgba(99, 102, 241, 0.1);
  border: 1px solid rgba(99, 102, 241, 0.3);
  border-radius: 8px;
  color: #a5b4fc;
  font-size: 0.825rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.showcase-card:hover .apply-btn {
  background: var(--primary-gradient);
  border-color: transparent;
  color: white;
}

.arrow-icon {
  width: 14px;
  height: 14px;
  transition: transform 0.2s ease;
}

.apply-btn:hover .arrow-icon {
  transform: translateX(3px);
}
</style>
