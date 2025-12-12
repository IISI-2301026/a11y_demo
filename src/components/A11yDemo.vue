<template>
  <div :class="[contrastClass, grayscaleClass, largeTextClass]">
    <a href="#main-content" class="skip-link">跳到主要內容</a>
    <header>
      <h1>🌐 無障礙網頁實務演示</h1>
      <p class="subtitle">讓你的網站成為所有人的好朋友</p>
    </header>
    <nav role="navigation" aria-label="主要導航">
      <div class="nav-container">
        <ul class="nav-list">
          <li><a href="#forms" accesskey="1" title="Alt+1 跳到表單範例">表單範例 <span class="sr-only">(Alt+1)</span></a></li>
          <li><a href="#buttons" accesskey="2" title="Alt+2 跳到按鈕範例">按鈕範例 <span class="sr-only">(Alt+2)</span></a></li>
          <li><a href="#contrast" accesskey="3" title="Alt+3 跳到色彩對比">色彩對比 <span class="sr-only">(Alt+3)</span></a></li>
          <li><a href="#keyboard" accesskey="4" title="Alt+4 跳到鍵盤導航">鍵盤導航 <span class="sr-only">(Alt+4)</span></a></li>
          <li><a href="#aria" accesskey="5" title="Alt+5 跳到ARIA範例">ARIA範例 <span class="sr-only">(Alt+5)</span></a></li>
          <li><a href="#modern" accesskey="6" title="Alt+6 跳到現代技術">現代技術 <span class="sr-only">(Alt+6)</span></a></li>
          <li><a href="#testing" accesskey="7" title="Alt+7 跳到測試工具">測試工具 <span class="sr-only">(Alt+7)</span></a></li>
        </ul>
      </div>
    </nav>
    <main id="main-content">
      <!-- 表單範例 -->
      <section id="forms" class="demo-section">
        <h2>📝 表單無障礙範例</h2>
        <div class="example-grid">
          <div class="example-card">
            <h3>✅ 好的表單設計</h3>
            <form @submit.prevent>
              <div class="form-group">
                <label for="name">姓名 <span aria-label="必填" style="color: #B30000;">*</span></label>
                <input type="text" id="name" v-model="form.name" required aria-required="true"
                  aria-describedby="name-help" />
                <div id="name-help" class="sr-only">請輸入您的真實姓名</div>
              </div>
              <div class="form-group">
                <label for="email">電子郵件 <span aria-label="必填" style="color: #B30000;">*</span></label>
                <input type="email" id="email" v-model="form.email" required aria-required="true"
                  aria-describedby="email-error" @blur="validateEmail"
                  :aria-invalid="emailInvalid ? 'true' : 'false'" />
                <div id="email-error" class="error-message" v-show="emailInvalid">請輸入有效的電子郵件格式</div>
              </div>
              <div class="form-group">
                <label for="department">團隊</label>
                <select id="department" v-model="form.department">
                  <option value="">請選擇團隊</option>
                  <option value="frontend">前端開發</option>
                  <option value="backend">後端開發</option>
                  <option value="design">UIUX設計</option>
                </select>
              </div>
            </form>
          </div>
          <div class="example-card">
            <h3>❌ 不好的表單設計</h3>
            <p style="font-size: 0.9rem; color: #dc2626; margin-bottom: 1rem;">
              只用placeholder作提示，無法被螢幕報讀軟體正確識別為標籤
            </p>
            <form>
              <div class="form-group">
                <input type="text" placeholder="姓名*" style="border: 1px solid #ccc;" />
              </div>
              <div class="form-group">
                <input type="text" placeholder="Email" style="border: 1px solid #ccc;" />
                <div style="color: #999; font-size: 0.9rem;">格式錯誤</div>
              </div>
              <div class="form-group">
                <select style="border: 1px solid #ccc;">
                  <option>部門</option>
                  <option>前端</option>
                  <option>後端</option>
                </select>
              </div>
            </form>
          </div>
        </div>
      </section>
      <!-- 按鈕範例 -->
      <section id="buttons" class="demo-section">
        <h2>🔘 按鈕與互動元素</h2>
        <div class="example-grid">
          <div class="example-card">
            <h3>✅ 語意化按鈕</h3>
            <div class="button-group">
              <button class="btn btn-primary" type="submit">提交表單</button>
              <button class="btn btn-secondary" type="button" @click="showAlert('這是一個可訪問的按鈕！')">取消</button>
              <button class="btn btn-primary" type="button" :aria-expanded="menuExpanded ? 'true' : 'false'"
                aria-controls="menu-1" @click="toggleMenu">
                選單 ▼
              </button>
            </div>
          </div>
          <div class="example-card">
            <h3>❌ 不好的按鈕設計</h3>
            <div class="button-group">
              <div class="btn btn-primary" style="cursor: pointer;">提交</div>
              <span class="btn btn-secondary" style="cursor: pointer;">取消</span>
              <div style="background: #2563eb; color: white; padding: 0.75rem; cursor: pointer;">點我</div>
            </div>
          </div>
        </div>
      </section>
      <!-- 色彩對比範例 -->
      <section id="contrast" class="demo-section">
        <h2>🎨 色彩對比範例</h2>
        <div class="contrast-examples">
          <div class="contrast-good">
            <h3>✅ 良好對比度 (AA級別)</h3>
            <p>這段文字有足夠的對比度，符合WCAG 2.1 AA標準。對比度比例為 4.5:1 以上，確保大多數用戶都能輕鬆閱讀。</p>
          </div>
          <div class="contrast-bad">
            <h3>❌ 對比度不足</h3>
            <p>這段文字的對比度不足，可能對視力不佳或色覺障礙的用戶造成閱讀困難。</p>
          </div>
        </div>
      </section>
      <!-- 鍵盤導航範例 -->
      <section id="keyboard" class="demo-section">
        <h2>⌨️ 鍵盤導航測試區</h2>
        <p><strong>試試看：</strong>使用 Tab 鍵瀏覽下面的元素，觀察焦點指示器</p>
        <div class="focus-demo">
          <button class="btn btn-primary">按鈕 1</button>
          <a href="#" class="btn btn-secondary">連結按鈕</a>
          <input type="text" placeholder="輸入框" aria-label="輸入框" style="margin: 0 10px; padding: 8px;" />
          <select>
            <option>選項 1</option>
            <option>選項 2</option>
          </select>
        </div>
      </section>
      <!-- ARIA範例 -->
      <section id="aria" class="demo-section">
        <h2>♿ ARIA 標籤與動態內容</h2>
        <div class="example-grid">
          <div class="example-card">
            <h3>動態內容更新</h3>
            <button class="btn btn-primary" @click="updateStatus">更新狀態</button>
            <div id="status" class="live-region" aria-live="polite" aria-atomic="true">
              {{ statusText }}
            </div>
          </div>
          <div class="example-card">
            <h3>可展開面板</h3>
            <button class="btn btn-secondary" @click="togglePanel" :aria-expanded="panelExpanded ? 'true' : 'false'"
              aria-controls="expandable-panel">
              {{ panelExpanded ? '收起詳細資訊' : '展開詳細資訊' }}
            </button>
            <div id="expandable-panel" v-show="panelExpanded"
              style="margin-top: 1rem; padding: 1rem; background: #f0f9ff; border-radius: 4px;">
              <p>這是可展開的內容區域。螢幕報讀軟體會知道這個面板的展開狀態。</p>
            </div>
          </div>
        </div>
      </section>
    </main>

    <!-- 現代無障礙技術 -->
    <section id="modern">
      <ModernA11yFeatures />
    </section>

    <!-- 測試工具 -->
    <section id="testing">
      <A11yTestingTools />
    </section>

    <!-- 輔助工具面板 -->
    <div class="tools-panel">
      <h4>🛠️ 無障礙輔助工具</h4>
      <!-- <button class="tool-btn" @click="toggleHighContrast">高對比模式</button> -->
      <button class="tool-btn" @click="toggleGrayscale">灰階模式</button>
      <button class="tool-btn" @click="toggleLargeText">大字體模式</button>
      <button class="tool-btn" @click="simulateScreenReader">模擬螢幕報讀</button>
    </div>

    <!-- 參考資料與相關資源 -->
    <footer class="references-section">
      <div class="references-container">
        <h2>📚 參考資料與相關資源</h2>

        <div class="references-grid">
          <!-- 官方標準與指南 -->
          <div class="reference-card">
            <h3>🌍 官方標準與指南</h3>
            <ul class="reference-list">
              <li>
                <a href="https://www.w3.org/WAI/WCAG21/quickref/" target="_blank" rel="noopener noreferrer">
                  WCAG 2.1 快速參考指南
                </a>
                <p>W3C 網頁內容無障礙指南的官方快速參考</p>
              </li>
              <li>
                <a href="https://www.w3.org/WAI/ARIA/apg/" target="_blank" rel="noopener noreferrer">
                  ARIA Authoring Practices Guide
                </a>
                <p>ARIA 最佳實作範例與設計模式</p>
              </li>
              <li>
                <a href="https://developer.mozilla.org/en-US/docs/Web/Accessibility" target="_blank"
                  rel="noopener noreferrer">
                  MDN Web Docs - Accessibility
                </a>
                <p>Mozilla 開發者網路的無障礙開發指南</p>
              </li>
            </ul>
          </div>

          <!-- 測試工具 -->
          <div class="reference-card">
            <h3>🧪 測試工具與檢查器</h3>
            <ul class="reference-list">
              <li>
                <a href="https://chromewebstore.google.com/detail/lhdoppojpmngadmnindnejefpokejbdd?utm_source=item-share-cb"
                  target="_blank" rel="noopener noreferrer">
                  axe DevTools
                </a>
                <p>自動化無障礙測試工具，瀏覽器擴充套件</p>
              </li>
              <li>
                <a href="https://chromewebstore.google.com/detail/jbbplnpkjmmeebjpijfedlgcdilocofh?utm_source=item-share-cb"
                  target="_blank" rel="noopener noreferrer">
                  WAVE Web Accessibility Evaluator
                </a>
                <p>WebAIM 提供的免費網頁無障礙評估工具</p>
              </li>
              <li>
                <a href="https://www.tpgi.com/color-contrast-checker/" target="_blank" rel="noopener noreferrer">
                  Color Contrast Checker
                </a>
                <p>色彩對比度檢測工具</p>
              </li>
              <li>
                <a href="https://developer.chrome.com/docs/lighthouse/overview?hl=zh-tw" target="_blank" rel="noopener noreferrer">
                  Lighthouse 網頁無障礙檢測
                </a>
                <p>Google 官方的網站效能與無障礙自動化檢測工具</p>
              </li>
            </ul>
          </div>

          <!-- 螢幕報讀器資源 -->
          <div class="reference-card">
            <h3>🔊 螢幕報讀器與輔助技術</h3>
            <ul class="reference-list">
              <li>
                <a href="https://www.nvaccess.org/" target="_blank" rel="noopener noreferrer">
                  NVDA Screen Reader
                </a>
                <p>免費開源螢幕報讀軟體</p>
              </li>
            </ul>
          </div>

          <!-- 政府相關無障礙資源 -->
          <div class="reference-card">
            <h3>🇹🇼 政府相關無障礙資源</h3>
            <ul class="reference-list">
              <li>
                <a href="https://accessibility.ncc.gov.tw/" target="_blank" rel="noopener noreferrer">
                  國家通訊傳播委員會 - 網站無障礙規範
                </a>
                <p>台灣政府網站無障礙規範與檢測標準</p>
              </li>
              <li>
                <a href="https://www.handicap-free.nat.gov.tw/" target="_blank" rel="noopener noreferrer">
                  身心障礙者服務資訊網
                </a>
                <p>政府身心障礙者相關服務與資訊</p>
              </li>
              <li>
                <a href="https://freego.sfaa.gov.tw/" target="_blank" rel="noopener noreferrer">
                  無障礙環境資訊網
                </a>
                <p>台灣無障礙環境相關資訊與資源</p>
              </li>
            </ul>
          </div>

          <!-- 其他資源 -->
          <div class="reference-card">
            <h3>⚡ 其他無障礙資源</h3>
            <ul class="reference-list">
              <li>
                <a href="https://vuejs.org/guide/best-practices/accessibility.html" target="_blank"
                  rel="noopener noreferrer">
                  Vue.js Accessibility Guide
                </a>
                <p>Vue.js 官方無障礙開發指南</p>
              </li>
            </ul>
          </div>
        </div>

      </div>
    </footer>
  </div>
</template>

<script>
import { ref, reactive } from 'vue';
import ModernA11yFeatures from './ModernA11yFeatures.vue';
import A11yTestingTools from './A11yTestingTools.vue';

export default {
  name: 'A11yDemo',
  components: {
    ModernA11yFeatures,
    A11yTestingTools
  },
  setup() {
    // 響應式數據
    const form = reactive({
      name: '',
      email: '',
      department: ''
    });

    const emailInvalid = ref(false);
    const menuExpanded = ref(false);
    const statusCount = ref(0);
    const statusText = ref('狀態：等待更新...');
    const panelExpanded = ref(false);
    const contrastClass = ref('');
    const grayscaleClass = ref('');
    const largeTextClass = ref('');

    // 方法
    const validateEmail = () => {
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      emailInvalid.value = form.email && !emailRegex.test(form.email);
    };

    const showAlert = (msg) => {
      alert(msg);
    };

    const toggleMenu = () => {
      menuExpanded.value = !menuExpanded.value;
    };

    const updateStatus = () => {
      statusCount.value++;
      statusText.value = `狀態：已更新 ${statusCount.value} 次 - ${new Date().toLocaleTimeString()}`;
    };

    const togglePanel = () => {
      panelExpanded.value = !panelExpanded.value;
    };

    const toggleHighContrast = () => {
      contrastClass.value = contrastClass.value ? '' : 'high-contrast';
    };

    const toggleGrayscale = () => {
      grayscaleClass.value = grayscaleClass.value ? '' : 'grayscale';
    };

    const toggleLargeText = () => {
      largeTextClass.value = largeTextClass.value ? '' : 'large-text';
    };

    const simulateScreenReader = () => {
      const elements = document.querySelectorAll('h1, h2, h3, button, a, input, label');
      let index = 0;
      const statusEl = document.getElementById('status');
      function readNext() {
        if (index < elements.length) {
          const element = elements[index];
          element.focus();
          const text = element.textContent || element.getAttribute('aria-label') || element.getAttribute('alt') || '互動元素';
          if (statusEl) statusEl.textContent = `螢幕報讀：${text}`;
          index++;
          setTimeout(readNext, 1500);
        }
      }
      readNext();
    };

    return {
      form,
      emailInvalid,
      menuExpanded,
      statusCount,
      statusText,
      panelExpanded,
      contrastClass,
      grayscaleClass,
      largeTextClass,
      validateEmail,
      showAlert,
      toggleMenu,
      updateStatus,
      togglePanel,
      toggleHighContrast,
      toggleGrayscale,
      toggleLargeText,
      simulateScreenReader
    };
  }
};
</script>


<style scoped>
/* 參考資料區塊樣式 */
.references-section {
  background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
  padding: 3rem 0;
  margin-top: 4rem;
  border-top: 3px solid #3b82f6;
}

.references-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 2rem;
}

.references-section h2 {
  text-align: center;
  color: #1e293b;
  margin-bottom: 3rem;
  font-size: 2.5rem;
  font-weight: 700;
}

.references-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 2rem;
  margin-bottom: 3rem;
}

.reference-card {
  background: white;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.05), 0 1px 3px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e7eb;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.reference-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05);
}

.reference-card h3 {
  color: #1e293b;
  font-size: 1.25rem;
  font-weight: 600;
  margin-bottom: 1.5rem;
  padding-bottom: 0.75rem;
  border-bottom: 2px solid #e5e7eb;
}

.reference-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.reference-list li {
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid #f1f5f9;
}

.reference-list li:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.reference-list a {
  color: #3b82f6;
  text-decoration: none;
  font-weight: 600;
  font-size: 1rem;
  transition: color 0.2s ease;
  display: inline-block;
  margin-bottom: 0.5rem;
}

.reference-list a:hover {
  color: #1d4ed8;
  text-decoration: underline;
}

.reference-list a:focus {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
  border-radius: 2px;
}

.reference-list p {
  color: #64748b;
  font-size: 0.875rem;
  line-height: 1.5;
  margin: 0;
}


.update-info {
  font-size: 0.875rem;
  color: #64748b;
  font-style: italic;
}

/* 響應式設計 */
@media (max-width: 768px) {
  .references-section {
    padding: 2rem 0;
  }

  .references-container {
    padding: 0 1rem;
  }

  .references-section h2 {
    font-size: 2rem;
    margin-bottom: 2rem;
  }

  .references-grid {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }

  .reference-card {
    padding: 1.5rem;
  }
}

/* 高對比模式支援 */
.high-contrast .reference-card {
  border: 2px solid #000;
  background: #fff;
}

.high-contrast .reference-list a {
  color: #0000ff;
}

.high-contrast .reference-list a:visited {
  color: #800080;
}

/* 大字體模式支援 */
.large-text .reference-list a {
  font-size: 1.125rem;
}

.large-text .reference-list p {
  font-size: 1rem;
}

.large-text .references-section h2 {
  font-size: 3rem;
}
</style>
