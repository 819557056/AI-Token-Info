<template>
  <div id="app" class="container">
    <header class="input-section">
      <div class="input-group">
        <label for="inputTokens">输入Token:</label>
        <input type="number" id="inputTokens" v-model.number="inputTokens" placeholder="0">
      </div>
      <div class="input-group">
        <label for="outputTokens">输出Token:</label>
        <input type="number" id="outputTokens" v-model.number="outputTokens" placeholder="0">
      </div>
    </header>

    <main class="content-section">
      <div class="content-block">
        <h2>one-hub</h2>
        <div class="details">
          <p>原输入价格: $1.25 /M</p>
          <p>原输出价格: $10 /M</p>
          <p>分组折扣: 2.00 倍</p>
          <hr>
          <p>实际输入价格：${{ oneHubActualInputPrice.toFixed(6) }}</p>
          <p>实际输出价格：${{ oneHubActualOutputPrice.toFixed(6) }}</p>
          <h3>最终价格：${{ oneHubTotalPrice.toFixed(6) }}</h3>
        </div>
      </div>

      <div class="content-block">
        <h2>new-api</h2>
        <div class="details">
          <p>模型倍率: 1.25</p>
          <p>补全倍率: 4</p>
          <p>分组倍率: 1</p>
          <p>提示价格：$2.5 / 1M tokens</p>
          <p>补全价格：$2.5 * 4 = $10 / 1M tokens (补全倍率: 4)</p>
          <hr>
          <p>实际输入价格：${{ newApiActualInputPrice.toFixed(6) }}</p>
          <p>实际输出价格：${{ newApiActualOutputPrice.toFixed(6) }}</p>
          <h3>最终价格：${{ newApiTotalPrice.toFixed(6) }}</h3>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      inputTokens: 0,
      outputTokens: 0,
    };
  },
  computed: {
    // One-Hub Calculations
    oneHubActualInputPrice() {
      // 实际输入价格：计算公式：( 输入Toekn数量 / 1000000 * 2.5) -> Assuming original price was $1.25, and discount is 2x, so effective price is $1.25 * 2 = $2.5
      // The user stated: 原输入价格: $1.25 /M and 分组折扣: 2.00 倍.
      // The formula provided was: ( 输入Toekn数量 / 1000000 * 2.5)
      // This implies the $2.5 in the formula is the discounted price per million tokens.
      return (this.inputTokens / 1000000) * 1.25 * 2.00; // Corrected based on "原输入价格" and "分组折扣"
    },
    oneHubActualOutputPrice() {
      // 实际输出价格：计算公式：(输出Toekn数量 / 1000000 * 20) -> Assuming original price was $10, and discount is 2x, so effective price is $10 * 2 = $20
      // The user stated: 原输出价格: $10 /M and 分组折扣: 2.00 倍.
      // The formula provided was: (输出Toekn数量 / 1000000 * 20)
      // This implies the $20 in the formula is the discounted price per million tokens.
      return (this.outputTokens / 1000000) * 10 * 2.00; // Corrected based on "原输出价格" and "分组折扣"
    },
    oneHubTotalPrice() {
      return this.oneHubActualInputPrice + this.oneHubActualOutputPrice;
    },

    // New-API Calculations
    newApiActualInputPrice() {
      // 实际输入价格：计算公式：输入Toekn数量 / 1M tokens * $2.5
      const pricePerMillion = 2.5;
      return (this.inputTokens / 1000000) * pricePerMillion;
    },
    newApiActualOutputPrice() {
      // 实际输出价格：计算公式：输出Toekn数量 / 1M tokens * $10 * 分组 1
      // 补全价格：$2.5 * 4 = $10 / 1M tokens (补全倍率: 4)
      // 分组倍率: 1
      const completionPricePerMillion = 2.5 * 4; // $10
      const groupMultiplier = 1;
      return (this.outputTokens / 1000000) * completionPricePerMillion * groupMultiplier;
    },
    newApiTotalPrice() {
      return this.newApiActualInputPrice + this.newApiActualOutputPrice;
    }
  }
};
</script>

<style>
.container {
  max-width: 1000px;
  margin: 20px auto;
  padding: 20px;
  font-family: sans-serif;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.input-section {
  display: flex;
  flex-direction: column; /* Stack input groups vertically */
  align-items: center; /* Center the groups */
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 1px solid #eee;
}

.input-group {
  display: flex;
  flex-direction: row; /* Align label and input horizontally */
  align-items: center; /* Vertically center label and input */
  margin-bottom: 10px; /* Add some space between the two input groups */
}

.input-group label {
  margin-right: 10px; /* Add space between label and input */
  font-weight: bold;
  color: #333;
}

.input-group input {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1em;
  width: 200px;
}

.content-section {
  display: flex;
  justify-content: space-between;
  gap: 20px; /* Adds space between the two content blocks */
}

.content-block {
  flex: 1; /* Each block takes equal width */
  background-color: #fff;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.05);
}

.content-block h2 {
  text-align: center;
  margin-top: 0;
  margin-bottom: 15px;
  color: #007bff;
  border-bottom: 2px solid #007bff;
  padding-bottom: 10px;
}

.content-block .details p {
  margin: 10px 0;
  line-height: 1.6;
  color: #555;
}

.content-block .details hr {
  margin: 15px 0;
  border: 0;
  border-top: 1px dashed #ccc;
}

.content-block .details h3 {
  margin-top: 15px;
  color: #28a745;
  text-align: right;
  font-size: 1.2em;
}

/* Basic responsive behavior */
@media (max-width: 768px) {
  .input-section {
    flex-direction: column;
    align-items: center;
  }
  .input-group {
    width: 80%;
    margin-bottom: 15px;
  }
  .input-group input {
    width: 100%;
  }
  .content-section {
    flex-direction: column;
  }
  .content-block {
    margin-bottom: 20px;
  }
}
</style>
