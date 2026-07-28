<template>
  <div class="app-container">
    <!-- 成功提示 -->
    <div v-if="copied" class="top-success-toast">
      复制成功
    </div>
    <!-- 常驻悬浮分享按钮 -->
    <button class="floating-share-btn" @click="showShareGuide = true">
      <svg class="share-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <circle cx="18" cy="5" r="3"></circle>
        <circle cx="6" cy="12" r="3"></circle>
        <circle cx="18" cy="19" r="3"></circle>
        <line x1="8.59" y1="13.51" x2="15.42" y2="17.49"></line>
        <line x1="15.41" y1="6.51" x2="8.59" y2="10.49"></line>
      </svg>
      <span>分享公文神器</span>
    </button>

    <header>
      <h1>{{ appTitle }}</h1>
      <p>党政机关标准公文 · 年度/季度述职报告 · 领导讲话稿与发言 · 专项整改与督查报告</p>
    </header>

    <!-- 动态广播轮播 -->
    <UserTicker />

    <!-- 核心操作区卡片 -->
    <main ref="inputCardRef" class="glass-card input-group">
      <!-- 4 种预设类型选择 -->
      <div class="selector-group">
        <label class="selector-label">选择公文汇报类型</label>
        <div class="style-selector">
          <button 
            v-for="gtype in gongwenTypeOptions" 
            :key="gtype"
            class="style-option"
            :class="{ active: activeGongwenType === gtype }"
            @click="activeGongwenType = gtype"
          >
            {{ gtype }}
          </button>
        </div>
      </div>

      <!-- 2 组属性：适用单位 & 字体文号规范 -->
      <div class="options-row" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem;">
        <div class="selector-group">
          <label class="selector-label">适用单位类型</label>
          <div class="style-selector">
            <button 
              v-for="org in orgTypeOptions" 
              :key="org"
              class="style-option"
              :class="{ active: selectedOrgType === org }"
              @click="selectedOrgType = org"
            >
              {{ org }}
            </button>
          </div>
        </div>

        <div class="selector-group">
          <label class="selector-label">字体文号规范</label>
          <div class="style-selector">
            <button 
              v-for="format in formatTierOptions" 
              :key="format"
              class="style-option"
              :class="{ active: selectedFormatTier === format }"
              @click="selectedFormatTier = format"
            >
              {{ format }}
            </button>
          </div>
        </div>
      </div>

      <!-- 输入框 -->
      <div class="selector-group">
        <div style="display: flex; justify-content: space-between; align-items: center;">
          <label class="selector-label">输入公文/工作汇报的主题或核心要点</label>
          <div style="display: flex; gap: 0.5rem;">
            <button v-if="userInput" class="text-link-btn" @click="userInput = ''">清空输入</button>
            <button class="text-link-btn" @click="showGongwenRulesModal = true">公文处理条例与避坑指南</button>
          </div>
        </div>
        <textarea 
          v-model="userInput" 
          placeholder="请简要描述您的公文主题或汇报要点...（例如：关于开展2025年二季度消防安全大检查的请示，需包含隐排查对象、检查分组、整改要求及经费预算。）"
          style="min-height: 120px;"
        ></textarea>
        <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-secondary);">
          <span>字符数: {{ userInput.length }} 字</span>
          <span>建议明确主旨背景、核心成绩或整改举措</span>
        </div>
      </div>

      <!-- 操作按钮区 -->
      <div style="display: flex; gap: 0.75rem;">
        <button 
          class="action-btn" 
          :disabled="loading || !userInput.trim()"
          @click="handleGenerate"
        >
          {{ loading ? '正在精准生成严谨规范的公文与汇报中...' : '一键生成标准公文/汇报材料' }}
        </button>
        <button class="icon-btn" style="padding: 0 1rem; border-radius: 10px;" @click="toggleHistoryDrawer">
          历史方案 ({{ historyList.length }})
        </button>
      </div>

      <!-- 异常提示 -->
      <div v-if="errorMsg" style="color: var(--accent-color); font-size: 0.85rem; text-align: center; margin-top: 0.5rem;">
        {{ errorMsg }}
      </div>
    </main>

    <!-- 生成结果卡片 -->
    <section v-if="result || loading" class="glass-card">
      <div class="result-header">
        <span class="result-title">标准公文/工作汇报生成结果</span>
        <div class="button-actions">
          <button v-if="result" class="icon-btn" @click="copyText">
            {{ copied ? '已复制公文' : '复制公文材料' }}
          </button>
          <button v-if="result" class="icon-btn" @click="resetResult">
            重置
          </button>
        </div>
      </div>

      <!-- 加载中骨架屏 -->
      <div v-if="loading" class="skeleton">
        <div class="skeleton-line" style="width: 85%"></div>
        <div class="skeleton-line" style="width: 95%"></div>
        <div class="skeleton-line" style="width: 70%"></div>
        <div class="skeleton-line" style="width: 90%"></div>
        <div class="skeleton-line" style="width: 60%"></div>
      </div>

      <!-- 渲染结果 -->
      <div v-else-if="result">
        <!-- AI 共识打分可视化看板 -->
        <div v-if="aiScores" class="scores-container" style="margin-bottom: 1.5rem; padding: 1.25rem; background: rgba(0,0,0,0.25); border-radius: 12px; border: 1px solid rgba(255,255,255,0.06);">
          <div style="font-weight: 700; font-size: 0.95rem; margin-bottom: 1rem; color: #a5b4fc; display: flex; justify-content: space-between; align-items: center;">
            <span>AI 公文质量与规范评估看板</span>
            <span style="font-size: 0.8rem; font-weight: normal; color: var(--text-secondary);">综合公文规范分: {{ getAverageScoreFromMap(aiScores) }} / 100</span>
          </div>
          <div class="metrics-grid" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1rem;">
            <div v-for="metric in metricsList" :key="metric.key" class="metric-item" style="background: rgba(255,255,255,0.03); padding: 0.75rem; border-radius: 8px; border: 1px solid rgba(255,255,255,0.04);">
              <div style="font-size: 0.775rem; color: var(--text-secondary); margin-bottom: 0.35rem;">{{ metric.label }}</div>
              <div style="display: flex; justify-content: space-between; align-items: center;">
                <div style="font-size: 1.1rem; font-weight: 700; color: #6366f1;">{{ aiScores[metric.key] || 90 }} <span style="font-size: 0.7rem; color: var(--text-secondary);">分</span></div>
                <div class="score-bar" style="width: 45px; height: 6px; background: rgba(255,255,255,0.1); border-radius: 3px; overflow: hidden;">
                  <div class="score-fill" :style="{ width: (aiScores[metric.key] || 90) + '%', background: 'var(--primary-gradient)', height: '100%' }"></div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="output-content" style="white-space: pre-wrap; line-height: 1.7; font-size: 0.925rem; color: var(--text-primary);">{{ displayResultText }}</div>
      </div>
    </section>

    <!-- PC端 Nomads 案例与模版展示 -->
    <NomadsShowcase
      @apply-template="handleApplyTemplate"
    />

    <!-- Toast 浮层提示 -->
    <div v-if="toastVisible" class="toast-notification">
      {{ toastMessage }}
    </div>

    <!-- H5 / 桌面端 分享弹窗 -->
    <div v-if="showShareGuide" class="modal-overlay" @click.self="showShareGuide = false">
      <div class="modal-content">
        <h3>分享网腾无限AI公文专家</h3>
        <div class="modal-text-content">
          <p>将本应用分享给身边有体制内公文撰写、述职总结与会议发言需求的同事或伙伴！</p>
          <div style="background: rgba(255,255,255,0.04); padding: 0.75rem; border-radius: 8px; border: 1px solid var(--card-border); margin: 0.75rem 0; font-size: 0.85rem; color: #a5b4fc;">
            https://ai.wuxian.xyz/ai-gongwen
          </div>
        </div>
        <div style="display: flex; gap: 0.5rem; justify-content: flex-end;">
          <button class="modal-btn" @click="copyShareLink">复制链接</button>
          <button class="modal-btn" style="background: transparent; border: 1px solid var(--card-border);" @click="showShareGuide = false">关闭</button>
        </div>
      </div>
    </div>

    <!-- 公文处理条例与避坑规范指南弹窗 -->
    <div v-if="showGongwenRulesModal" class="modal-overlay" @click.self="showGongwenRulesModal = false">
      <div class="modal-content" style="max-width: 580px;">
        <h3>党政机关公文处理工作条例与避坑指南</h3>
        <div class="modal-text-content modal-scroll-area">
          <div v-for="(rule, idx) in gongwenRules" :key="idx" style="margin-bottom: 1rem; padding-bottom: 0.75rem; border-bottom: 1px solid rgba(255,255,255,0.06);">
            <div style="font-weight: 600; color: #a5b4fc; margin-bottom: 0.25rem;">{{ idx + 1 }}. {{ rule.title }}</div>
            <div style="font-size: 0.825rem; color: var(--text-primary); margin-bottom: 0.25rem;">正确规范: {{ rule.advice }}</div>
            <div style="font-size: 0.825rem; color: #f87171;">常见误区: {{ rule.avoid }}</div>
          </div>
        </div>
        <button class="modal-btn" @click="showGongwenRulesModal = false">已知晓</button>
      </div>
    </div>

    <!-- 本地历史记录抽屉 -->
    <div v-if="showHistory" class="modal-overlay" @click.self="showHistory = false">
      <div class="modal-content history-drawer" style="max-width: 600px; max-height: 80vh; display: flex; flex-direction: column;">
        <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 1rem;">
          <h3>本地历史生成记录</h3>
          <button class="text-link-btn" v-if="historyList.length > 0" @click="clearHistory">清空历史</button>
        </div>
        <div v-if="historyList.length === 0" style="text-align: center; padding: 2rem; color: var(--text-secondary); font-size: 0.85rem;">
          暂无本地历史生成记录
        </div>
        <div v-else class="modal-scroll-area" style="flex: 1; overflow-y: auto; display: flex; flex-direction: column; gap: 0.75rem;">
          <div 
            v-for="item in historyList" 
            :key="item.id"
            class="history-card"
            style="background: rgba(255,255,255,0.03); padding: 0.85rem; border-radius: 10px; border: 1px solid rgba(255,255,255,0.06);"
          >
            <div style="display: flex; justify-content: space-between; font-size: 0.75rem; color: var(--text-secondary); margin-bottom: 0.4rem;">
              <span>{{ item.timestamp }} · {{ item.gongwenType }}</span>
              <span style="color: #a5b4fc;">评分: {{ getAverageScore(item) }}分</span>
            </div>
            <div style="font-size: 0.85rem; color: var(--text-primary); margin-bottom: 0.5rem; font-weight: 500; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden;">
              {{ item.input }}
            </div>
            <div style="display: flex; gap: 0.5rem; justify-content: flex-end;">
              <button class="text-link-btn" @click="applyHistory(item)">套用此需求</button>
              <button class="text-link-btn" @click="viewHistoryOutput(item)">查看全文</button>
            </div>
          </div>
        </div>
        <button class="modal-btn" style="margin-top: 1rem;" @click="showHistory = false">关闭抽屉</button>
      </div>
    </div>

    <!-- 底部隐私与服务条款链接 -->
    <footer class="footer-links">
      <button class="footer-link-btn" @click="showPrivacy = true">Privacy Policy</button>
      <button class="footer-link-btn" @click="showTerms = true">Terms of Service</button>
      <button class="footer-link-btn" @click="showContact = true">Contact Us</button>
      <a href="https://api.wuxian.xyz/sign-up?aff=OyRY" target="_blank" rel="noopener noreferrer" class="footer-link-btn">API 平台</a>
      <a href="https://www.kutuyun.com/aff/IPJKCKWF" target="_blank" rel="noopener noreferrer" class="footer-link-btn">酷兔云</a>
      <a href="https://bandwagonhost.com/aff.php?aff=48115" target="_blank" rel="noopener noreferrer" class="footer-link-btn">搬瓦工</a>
    </footer>

    <!-- 隐私政策弹窗 -->
    <div v-if="showPrivacy" class="modal-overlay" @click.self="showPrivacy = false">
      <div class="modal-content">
        <h3>Privacy Policy</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>我们非常重视您的隐私。您在本应用中输入的所有公文或汇报文本仅用于实时大模型生成，我们不会在服务器端进行永久存储或记录。</p>
          <p>为了记录您的免费额度，本应用会在您的浏览器本地（localStorage）记录试用次数与解锁状态。</p>
        </div>
        <button class="modal-btn" @click="showPrivacy = false">关闭</button>
      </div>
    </div>

    <!-- 服务条款弹窗 -->
    <div v-if="showTerms" class="modal-overlay" @click.self="showTerms = false">
      <div class="modal-content">
        <h3>Terms of Service</h3>
        <div class="modal-text-content modal-scroll-area">
          <p>欢迎使用我们的 AI 微应用服务。使用本应用即代表您同意并承诺遵守当地有关人工智能生成内容（AIGC）的法律法规。</p>
          <p>所有生成结果均由 AI 模型计算产生，本应用不对生成内容的准确性、完整性及合法性承担任何直接或间接法律责任。</p>
        </div>
        <button class="modal-btn" @click="showTerms = false">关闭</button>
      </div>
    </div>

    <!-- 联系我们弹窗 -->
    <div v-if="showContact" class="modal-overlay" @click.self="showContact = false">
      <div class="modal-content contact-modal-content">
        <h3>Contact Us</h3>
        <div class="modal-text-content contact-card-body">
          <p>如果您在使用过程中遇到任何问题，或有合作意向，可以通过以下方式联系我们：</p>
          <div class="contact-qr-container">
            <div class="contact-qr-card">
              <img :src="weixinImg" alt="微信交流" class="contact-qr-img" />
              <span class="contact-qr-label">微信交流</span>
            </div>
            <div class="contact-qr-card">
              <img :src="dingtalkImg" alt="钉钉联系" class="contact-qr-img" />
              <span class="contact-qr-label">钉钉联系</span>
            </div>
          </div>
          <p class="contact-email">反馈邮箱: <span style="color: var(--primary-color);">us@wuxian.xyz</span></p>
        </div>
        <button class="modal-btn" @click="showContact = false">关闭</button>
      </div>
    </div>

    <!-- 裂变拦截弹窗 -->
    <FissionModal 
      :visible="showFission" 
      :wechat-id="wechatId"
      @unlocked="handleUnlocked"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import UserTicker from './components/UserTicker.vue';
import FissionModal from './components/FissionModal.vue';
import NomadsShowcase from './components/NomadsShowcase.vue';
import appConfig from './config.json';
import weixinImg from '../asset/weixin.png';
import dingtalkImg from '../asset/dingtalk.png';

// 配置参数
const appTitle = ref(appConfig.title || '网腾无限AI - 体制内公文与工作汇报专家');
const wechatId = ref(appConfig.wechatId || 'ai_wuxian_xyz');
const promptTopic = ref(appConfig.promptTopic || '体制内公文与工作汇报专家');

const inputCardRef = ref<HTMLElement | null>(null);
const userInput = ref('');
const loading = ref(false);
const errorMsg = ref('');
const result = ref('');
const copied = ref(false);

const showFission = ref(false);
const showPrivacy = ref(false);
const showTerms = ref(false);
const showContact = ref(false);
const showShareGuide = ref(false);
const showGongwenRulesModal = ref(false);

const toastMessage = ref('');
const toastVisible = ref(false);

const showToast = (msg: string) => {
  toastMessage.value = msg;
  toastVisible.value = true;
  setTimeout(() => {
    toastVisible.value = false;
  }, 2500);
};

// 解析 Cookie
const getCookie = (name: string): string | null => {
  const nameEQ = name + "=";
  const ca = document.cookie.split(';');
  for (let i = 0; i < ca.length; i++) {
    let c = ca[i];
    while (c.charAt(0) === ' ') c = c.substring(1, c.length);
    if (c.indexOf(nameEQ) === 0) return c.substring(nameEQ.length, c.length);
  }
  return null;
};

// 用户登录状态
const userToken = ref(getCookie('wuxian_session'));
const isLoggedIn = computed(() => !!userToken.value);
const authUsesCount = ref(parseInt(localStorage.getItem('auth_uses') || '0', 10));

// 4 种预设类型
const gongwenTypeOptions = [
  '党政机关标准公文请示与批复',
  '年度/季度工作总结与述职报告',
  '领导讲话稿与会议发言材料',
  '专项检查排查与整改报告'
];
const activeGongwenType = ref(gongwenTypeOptions[0]);

// 2 组属性：适用单位 & 字体文号规范
const orgTypeOptions = ['党委政府机关', '事业单位', '国有企业', '基层社区'];
const selectedOrgType = ref('党委政府机关');

const formatTierOptions = ['严谨红头文件格式', '标准工作汇报格式', '会议纪要演讲格式'];
const selectedFormatTier = ref('严谨红头文件格式');

// 5 大评估指标
const metricsList = [
  { key: 'officialFormatStandard', label: '公文格式规范度' },
  { key: 'politicalIdeologyDepth', label: '政治站位与思想深度' },
  { key: 'logicHierarchyClarity', label: '结构层次条理性' },
  { key: 'wordingRigor', label: '用词严谨得体度' },
  { key: 'practicalOperability', label: '贯彻落实可行度' }
];

const aiScores = ref<Record<string, number> | null>(null);

// 历史记录定义
interface HistoryItem {
  id: string;
  timestamp: string;
  gongwenType: string;
  orgType: string;
  formatTier: string;
  input: string;
  aiScores: Record<string, number> | null;
  output: string;
}

const historyList = ref<HistoryItem[]>([]);
const showHistory = ref(false);

// 党政机关公文处理工作条例与避坑规范指南
const gongwenRules = [
  { 
    title: '拟文与文种合规规范', 
    advice: '准确区分“请示”、“报告”、“通知”与“函”，做到一文一事，主送与抄送机关层级对应。', 
    avoid: '严禁在报告中夹带请示事项，严禁越级行文或滥用拟文文种。' 
  },
  { 
    title: '政治站位与措辞得体', 
    advice: '准确引用政策标准用语，表述客观严谨、文字简练、讲求实效。', 
    avoid: '严禁出现政治敏感错误、口语化俗语或逻辑前后矛盾的模糊表述。' 
  },
  { 
    title: '结构层次与数据准确', 
    advice: '统一采用“一、（一）1. (1)”标准序号排版，成果总结附量化数据支撑。', 
    avoid: '切忌层级序号混乱、无数据支撑空喊口号或抄袭泛泛而谈的套话。' 
  }
];

// 计算纯结果文本 (剔除打分标签 [GONGWEN_SCORES])
const displayResultText = computed(() => {
  if (!result.value) return '';
  return result.value.replace(/\[GONGWEN_SCORES\][\s\S]*?\[\/GONGWEN_SCORES\]/g, '').trim();
});

// 解析打分标签
const parseAiScores = (rawText: string) => {
  const match = rawText.match(/\[GONGWEN_SCORES\](.*?)\[\/GONGWEN_SCORES\]/);
  if (!match) return null;
  const content = match[1];
  const scoresObj: Record<string, number> = {};
  content.split(',').forEach(item => {
    const [key, val] = item.split(':');
    if (key && val) {
      scoresObj[key.trim()] = parseInt(val.trim(), 10) || 90;
    }
  });
  return Object.keys(scoresObj).length > 0 ? scoresObj : null;
};

// 计算平均分
const getAverageScoreFromMap = (scores: Record<string, number>) => {
  const keys = Object.keys(scores);
  if (keys.length === 0) return '93.5';
  const sum = keys.reduce((acc, k) => acc + (scores[k] || 90), 0);
  return (sum / keys.length).toFixed(1);
};

const getAverageScore = (item: HistoryItem) => {
  if (!item.aiScores) return '93.5';
  return getAverageScoreFromMap(item.aiScores);
};

// 本地历史记录读取与保存
const loadHistory = () => {
  try {
    const raw = localStorage.getItem('gongwen_history_records');
    historyList.value = raw ? JSON.parse(raw) : [];
  } catch (e) {
    historyList.value = [];
  }
};

const saveHistory = () => {
  localStorage.setItem('gongwen_history_records', JSON.stringify(historyList.value));
};

const addHistoryRecord = () => {
  const newItem: HistoryItem = {
    id: Date.now().toString(),
    timestamp: new Date().toLocaleString(),
    gongwenType: activeGongwenType.value,
    orgType: selectedOrgType.value,
    formatTier: selectedFormatTier.value,
    input: userInput.value,
    aiScores: aiScores.value,
    output: result.value
  };
  historyList.value.unshift(newItem);
  if (historyList.value.length > 20) {
    historyList.value = historyList.value.slice(0, 20);
  }
  saveHistory();
};

const toggleHistoryDrawer = () => {
  loadHistory();
  showHistory.value = !showHistory.value;
};

const clearHistory = () => {
  historyList.value = [];
  localStorage.removeItem('gongwen_history_records');
  showToast('已清空本地历史记录');
};

const applyHistory = (item: HistoryItem) => {
  userInput.value = item.input;
  activeGongwenType.value = item.gongwenType;
  if (item.orgType) selectedOrgType.value = item.orgType;
  if (item.formatTier) selectedFormatTier.value = item.formatTier;
  showHistory.value = false;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const viewHistoryOutput = (item: HistoryItem) => {
  userInput.value = item.input;
  result.value = item.output;
  aiScores.value = item.aiScores;
  showHistory.value = false;
};

// 限制与额度检测
const isLimitReached = computed(() => {
  if (isLoggedIn.value) {
    return authUsesCount.value >= 15;
  }
  const uses = parseInt(localStorage.getItem('free_uses') || '0', 10);
  const shared = localStorage.getItem('shared_fission') === 'true';
  return uses >= 3 && !shared;
});

const apiEndpoint = import.meta.env.DEV
  ? '/api/local/generate'
  : (import.meta.env.VITE_API_ENDPOINT || 'https://api.wuxian.xyz/api/v1/generate');

const handleGenerate = async () => {
  if (isLimitReached.value) {
    showFission.value = true;
    return;
  }

  loading.value = true;
  errorMsg.value = '';
  result.value = '';
  aiScores.value = null;

  try {
    const response = await fetch(apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      credentials: 'include',
      body: JSON.stringify({
        taskType: 'text',
        prompt: `任务指导: ${promptTopic.value}\n【公文类型】: ${activeGongwenType.value}\n【适用单位】: ${selectedOrgType.value}\n【格式规范】: ${selectedFormatTier.value}\n【公文主题与核心要点】: ${userInput.value}`,
        style: activeGongwenType.value
      })
    });

    const data = await response.json();
    if (data.error) {
      errorMsg.value = data.error;
    } else {
      result.value = data.result;
      aiScores.value = parseAiScores(data.result);
      
      addHistoryRecord();

      if (isLoggedIn.value) {
        const nextAuthUses = authUsesCount.value + 1;
        localStorage.setItem('auth_uses', nextAuthUses.toString());
        authUsesCount.value = nextAuthUses;
      } else {
        const currentUses = parseInt(localStorage.getItem('free_uses') || '0', 10);
        localStorage.setItem('free_uses', (currentUses + 1).toString());
      }
    }
  } catch (err: any) {
    errorMsg.value = '请求接口失败，请检查网络或本地代理服务。';
  } finally {
    loading.value = false;
  }
};

const handleApplyTemplate = (payload: {
  prompt: string;
  gongwenType?: string;
  orgType?: string;
  formatTier?: string;
}) => {
  userInput.value = payload.prompt;
  if (payload.gongwenType) activeGongwenType.value = payload.gongwenType;
  if (payload.orgType) selectedOrgType.value = payload.orgType;
  if (payload.formatTier) selectedFormatTier.value = payload.formatTier;
  if (inputCardRef.value) {
    inputCardRef.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
  }
};

const handleUnlocked = () => {
  showFission.value = false;
  handleGenerate();
};

const resetResult = () => {
  result.value = '';
  aiScores.value = null;
};

const copyText = async () => {
  try {
    await navigator.clipboard.writeText(displayResultText.value);
    copied.value = true;
    showToast('已成功复制公文材料至剪贴板');
    setTimeout(() => {
      copied.value = false;
    }, 2000);
  } catch (err) {
    errorMsg.value = '复制失败，请手动选择复制。';
  }
};

const copyShareLink = async () => {
  try {
    await navigator.clipboard.writeText('https://ai.wuxian.xyz/ai-gongwen');
    showToast('分享链接已复制到剪贴板');
    showShareGuide.value = false;
  } catch (err) {
    showToast('复制失败，请手动复制链接');
  }
};

onMounted(() => {
  loadHistory();
});
</script>

<style scoped>
.text-link-btn {
  background: none;
  border: none;
  color: #a5b4fc;
  font-size: 0.775rem;
  cursor: pointer;
  transition: color 0.2s ease;
}
.text-link-btn:hover {
  color: var(--text-primary);
  text-decoration: underline;
}

.floating-share-btn {
  position: fixed;
  right: 1.5rem;
  bottom: 2rem;
  z-index: 99;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.65rem 1.1rem;
  background: var(--primary-gradient);
  color: white;
  border: none;
  border-radius: 30px;
  font-size: 0.85rem;
  font-weight: 600;
  box-shadow: 0 4px 15px rgba(99, 102, 241, 0.4);
  cursor: pointer;
  transition: all 0.25s ease;
}

.floating-share-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(99, 102, 241, 0.6);
}

.share-icon {
  width: 16px;
  height: 16px;
}

.toast-notification {
  position: fixed;
  bottom: 5rem;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(15, 23, 42, 0.9);
  color: #a5b4fc;
  border: 1px solid rgba(99, 102, 241, 0.4);
  padding: 0.6rem 1.25rem;
  border-radius: 20px;
  font-size: 0.85rem;
  z-index: 1000;
  box-shadow: 0 10px 25px rgba(0,0,0,0.5);
  backdrop-filter: blur(8px);
}
</style>
