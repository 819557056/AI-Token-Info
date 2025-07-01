<template>
  <el-container class="main-container" :style="containerStyle">
    <el-header height="auto" class="app-header" >
      <h1 class="title">Token 价格计算器</h1>
    </el-header>

    <el-main>
      <el-card class="input-card" shadow="always">
        <el-form label-position="top">
          <el-row :gutter="20">
            <el-col :xs="24" :sm="12">
              <el-form-item label="输入Token数量">
                <el-input-number v-model="inputTokens" :min="0" placeholder="请输入输入Token数量" controls-position="right" style="width: 100%;" />
              </el-form-item>
            </el-col>
            <el-col :xs="24" :sm="12">
              <el-form-item label="输出Token数量">
                <el-input-number v-model="outputTokens" :min="0" placeholder="请输入输出Token数量" controls-position="right" style="width: 100%;" />
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
      </el-card>

      <el-row :gutter="20" class="content-row">
        <!-- one-hub 卡片 -->
        <el-col :xs="24" :lg="12">
          <el-card class="calculator-card one-hub-card" shadow="hover">
            <template #header>
              <div class="card-header">
                <span>one-hub</span>
              </div>
            </template>
            <el-form label-position="top" label-width="150px">
              <el-row :gutter="15">
                <el-col :md="12">
                  <el-form-item label="原输入单价 ($/M)">
                    <el-input-number v-model="oneHub.inputPrice" :precision="2" :step="0.01" controls-position="right" style="width: 100%;" />
                  </el-form-item>
                </el-col>
                <el-col :md="12">
                  <el-form-item label="原输出单价 ($/M)">
                    <el-input-number v-model="oneHub.outputPrice" :precision="2" :step="0.01" controls-position="right" style="width: 100%;" />
                  </el-form-item>
                </el-col>
                <el-col :md="12">
                  <el-form-item label="分组折扣 (倍)">
                    <el-input-number v-model="oneHub.discount" :precision="2" :step="0.01" controls-position="right" style="width: 100%;" />
                  </el-form-item>
                </el-col>
              </el-row>
            </el-form>
            <el-divider />
            <div class="results">
              <p>实际输入价格: <el-tag type="primary" size="large">${{ oneHubInputActual }}</el-tag></p>
              <p class="formula">输入Token数量 / 1M × 原输入单价 × 分组折扣</p>
              <p>实际输出价格: <el-tag type="primary" size="large">${{ oneHubOutputActual }}</el-tag></p>
              <p class="formula">输出Token数量 / 1M × 原输出单价 × 分组折扣</p>
              <div class="final-price one-hub-final">
                最终价格：${{ oneHubFinal }}
              </div>
            </div>
          </el-card>
        </el-col>

        <!-- new-api 卡片 -->
        <el-col :xs="24" :lg="12">
          <el-card class="calculator-card new-api-card" shadow="hover">
            <template #header>
              <div class="card-header">
                <span>new-api</span>
              </div>
            </template>
            <el-form label-position="top" label-width="150px">
              <el-row :gutter="15">
                <el-col :md="12">
                  <el-form-item label="模型倍率">
                    <el-input-number v-model="newApi.modelRate" :precision="2" :step="0.01" controls-position="right" style="width: 100%;" />
                  </el-form-item>
                </el-col>
                <el-col :md="12">
                  <el-form-item label="补全倍率">
                    <el-input-number v-model="newApi.completionRate" :precision="2" :step="0.01" controls-position="right" style="width: 100%;" />
                  </el-form-item>
                </el-col>
                <el-col :md="12">
                  <el-form-item label="分组倍率">
                    <el-input-number v-model="newApi.groupRate" :precision="2" :step="0.01" controls-position="right" style="width: 100%;" />
                  </el-form-item>
                </el-col>
                <el-col :md="12">
                  <el-form-item label="提示单价 ($/M)">
                    <el-input-number v-model="newApi.promptPrice" :precision="2" :step="0.01" controls-position="right" style="width: 100%;" />
                  </el-form-item>
                </el-col>
                <el-col :md="12">
                  <el-form-item label="补全单价 ($/M)">
                    <el-input-number v-model="newApi.completionPrice" :precision="2" :step="0.01" controls-position="right" style="width: 100%;" />
                  </el-form-item>
                </el-col>
              </el-row>
            </el-form>
            <el-divider />
            <div class="results">
              <p>实际补全价格: <el-tag type="info" size="large">${{ actualCompletionPrice }}</el-tag></p>
              <p class="formula">补全单价 × 补全倍率</p>
              <p>实际输入价格: <el-tag type="danger" size="large">${{ newApiInputActual }}</el-tag></p>
              <p class="formula">输入Token数量 / 1M × 提示单价 × 模型倍率</p>
              <p>实际输出价格: <el-tag type="danger" size="large">${{ newApiOutputActual }}</el-tag></p>
              <p class="formula">输出Token数量 / 1M × 实际补全价格 × 分组倍率</p>
              <div class="final-price new-api-final">
                最终价格：${{ newApiFinal }}
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";
import { ElContainer, ElHeader, ElMain, ElCard, ElForm, ElFormItem, ElInputNumber, ElRow, ElCol, ElDivider, ElTag } from 'element-plus';

// 容器宽度计算
const windowWidth = ref(window.innerWidth);
const baseWidth = 1920; // 基准屏幕宽度
const baseContainerWidth = 1200; // 在基准宽度下的容器宽度

// 计算容器宽度
const containerWidth = computed(() => {
  // 计算比例，但确保最小宽度不低于内容需要的最小宽度
  const ratio = windowWidth.value / baseWidth;
  const calculatedWidth = Math.round(baseContainerWidth * ratio);
  // 确保容器宽度不小于内容最小需求
  return Math.max(calculatedWidth, 320); // 设置一个最小宽度
});

// 容器样式对象
const containerStyle = computed(() => {
  return {
    width: `${containerWidth.value}px`,
    margin: '0 auto'
  };
});

// 监听窗口大小变化
const handleResize = () => {
  windowWidth.value = window.innerWidth;
};

// 组件挂载和卸载时添加/移除事件监听
onMounted(() => {
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});

// token 数量
const inputTokens = ref(0);
const outputTokens = ref(0);

// one-hub 参数
const oneHub = ref({
  inputPrice: 1.25,
  outputPrice: 10,
  discount: 2.0,
});

// new-api 参数
const newApi = ref({
  modelRate: 1.25,
  completionRate: 4,
  groupRate: 1,
  promptPrice: 2.5,
  completionPrice: 10,
});

// one-hub 计算
const oneHubInputActual = computed(() =>
  ((inputTokens.value / 1_000_000) * oneHub.value.inputPrice * oneHub.value.discount).toFixed(4)
);
const oneHubOutputActual = computed(() =>
  ((outputTokens.value / 1_000_000) * oneHub.value.outputPrice * oneHub.value.discount).toFixed(4)
);
const oneHubFinal = computed(() =>
  (parseFloat(oneHubInputActual.value) + parseFloat(oneHubOutputActual.value)).toFixed(4)
);

// new-api 计算
const actualCompletionPrice = computed(() =>
  (newApi.value.completionPrice * newApi.value.completionRate).toFixed(4)
);
const newApiInputActual = computed(() =>
  ((inputTokens.value / 1_000_000) * newApi.value.promptPrice * newApi.value.modelRate).toFixed(4)
);
const newApiOutputActual = computed(() =>
  ((outputTokens.value / 1_000_000) * actualCompletionPrice.value * newApi.value.groupRate).toFixed(4)
);
const newApiFinal = computed(() =>
  (parseFloat(newApiInputActual.value) + parseFloat(newApiOutputActual.value)).toFixed(4)
);
</script>

<style scoped>
html, body {
  overflow-y: hidden !important; /* Attempt to hide body scrollbar */
  height: 100%;
  margin: 0;
  padding: 0;
}

.main-container {
  padding: 10px; /* Reduced padding */
  background-color: #f0f2f5;
  min-height: 100vh;
  display: flex; /* Added for better flex control */
  flex-direction: column; /* Added for better flex control */
}

.app-header {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px; /* Reduced margin */
  flex-shrink: 0; /* Prevent header from shrinking */
}

.title {
  font-size: 2.2rem; /* Slightly reduced font size */
  font-weight: bold;
  color: #303133; /* Element Plus primary text color */
  text-align: center;
}

.el-main { /* Target el-main component for flex behavior */
  flex-grow: 1;
  overflow-y: auto; /* Allow main content to scroll if needed, though goal is no scroll */
  padding-top: 10px !important; /* Adjust el-main's default padding */
  padding-bottom: 10px !important;
}

.input-card {
  margin-bottom: 15px; /* Reduced margin */
  border-radius: 12px;
}

.content-row {
  margin-bottom: 10px; /* Reduced margin */
}

.calculator-card {
  border-radius: 12px;
  /* height: 100%; Let's remove this to see if it helps with fitting content */
  display: flex; /* Added for better flex control within card */
  flex-direction: column; /* Added for better flex control within card */
}

.calculator-card > :deep(.el-card__body) {
  flex-grow: 1; /* Make card body take available space */
}


.one-hub-card .card-header span {
  font-size: 1.8rem;
  font-weight: bold;
  color: #409EFF; /* Element Plus primary color */
}

.new-api-card .card-header span {
  font-size: 1.8rem;
  font-weight: bold;
  color: #E6A23C; /* Element Plus warning color for differentiation */
}


.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1.5rem;
  font-weight: bold;
}

.results p {
  font-size: 0.9rem; /* Slightly reduced font size */
  color: #606266;
  margin-bottom: 6px; /* Reduced margin */
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.results p .el-tag {
  font-weight: bold;
}


.results .formula {
  font-size: 0.75rem; /* Slightly reduced font size */
  color: #909399;
  margin-top: -4px; /* Reduced margin */
  margin-bottom: 10px; /* Reduced margin */
  text-align: right;
}

.final-price {
  margin-top: 15px; /* Reduced margin */
  padding: 12px; /* Reduced padding */
  border-radius: 8px;
  text-align: center;
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
}

.one-hub-final {
  background: linear-gradient(135deg, #409EFF, #79bbff); /* Element Plus primary gradient */
  box-shadow: 0 4px 15px rgba(64, 158, 255, 0.3);
}

.new-api-final {
  background: linear-gradient(135deg, #E6A23C, #ebb563); /* Element Plus warning gradient */
  box-shadow: 0 4px 15px rgba(230, 162, 60, 0.3);
}

/* Ensure el-input-number controls are visible and aligned */
:deep(.el-input-number .el-input__inner) {
  text-align: left;
}

.main-container {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.el-main {
  flex: 1;
  width: 100%;
}

/* 确保Element Plus的行和其他容器组件也是100%宽度 */
.el-row, .el-col, .el-card, .el-form {
  width: 100%;
}

/* 确保表格和其他大型组件能适应容器 */
.el-table {
  width: 100% !important;
}
</style>
