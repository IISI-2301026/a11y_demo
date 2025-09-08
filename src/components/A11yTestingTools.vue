<template>
  <div class="a11y-testing-tools">
    <h2>🧪 無障礙測試工具</h2>

    <div class="tools-grid">
      <!-- 自動化測試 -->
      <div class="tool-card">
        <h3>⚡ 自動化檢測</h3>
        <button @click="runAxeTest" :disabled="testing" class="btn btn-primary">
          {{ testing ? '檢測中...' : '執行 Axe 檢測' }}
        </button>
        <div v-if="axeResults" class="test-results">
          <h4>檢測結果:</h4>
          <div class="result-summary">
            <span class="violations">❌ 違規: {{ axeResults.violations.length }}</span>
            <span class="passes">✅ 通過: {{ axeResults.passes.length }}</span>
            <span class="incomplete">⚠️ 需手動檢查: {{ axeResults.incomplete.length }}</span>
          </div>

          <div v-if="axeResults.violations.length > 0" class="violations-list">
            <h5>發現的問題:</h5>
            <div v-for="(violation, index) in axeResults.violations" :key="index" class="violation-item">
              <strong>{{ violation.help }}</strong>
              <p>{{ violation.description }}</p>
              <small>影響程度: {{ violation.impact }}</small>
              <details>
                <summary>受影響的元素 ({{ violation.nodes.length }})</summary>
                <pre v-for="(node, nodeIndex) in violation.nodes" :key="nodeIndex">{{ node.html }}</pre>
              </details>
            </div>
          </div>
        </div>
      </div>

      <!-- 色彩對比檢測器 */
      <div class="tool-card">
        <h3>🎨 色彩對比檢測</h3>
        <div class="color-contrast-tool">
          <div class="color-input-group">
            <label for="fg-color">前景色:</label>
            <input type="color" id="fg-color" v-model="foregroundColor" />
            <input type="text" v-model="foregroundColor" placeholder="#000000" />
          </div>
          <div class="color-input-group">
            <label for="bg-color">背景色:</label>
            <input type="color" id="bg-color" v-model="backgroundColor" />
            <input type="text" v-model="backgroundColor" placeholder="#ffffff" />
          </div>
          
          <div class="contrast-preview" :style="contrastPreviewStyle">
            示例文字 - 這是色彩對比預覽
          </div>
          
          <div class="contrast-results">
            <div class="contrast-ratio">
              對比度: <strong>{{ contrastRatio }}:1</strong>
            </div>
            <div class="wcag-compliance">
              <div :class="['compliance-item', wcagAA.normal ? 'pass' : 'fail']">
                WCAG AA 一般文字: {{ wcagAA.normal ? '✅ 通過' : '❌ 未通過' }}
              </div>
              <div :class="['compliance-item', wcagAA.large ? 'pass' : 'fail']">
                WCAG AA 大文字: {{ wcagAA.large ? '✅ 通過' : '❌ 未通過' }}
              </div>
              <div :class="['compliance-item', wcagAAA.normal ? 'pass' : 'fail']">
                WCAG AAA 一般文字: {{ wcagAAA.normal ? '✅ 通過' : '❌ 未通過' }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 鍵盤導航測試 -->
      <div class="tool-card">
        <h3>⌨️ 鍵盤導航測試</h3>
        <div class="keyboard-test">
          <button @click="startKeyboardTest" class="btn btn-secondary">
            開始鍵盤導航測試
          </button>
          <div v-if="keyboardTestActive" class="keyboard-instructions">
            <p>使用以下按鍵測試導航:</p>
            <ul>
              <li><kbd>Tab</kbd> - 下一個元素</li>
              <li><kbd>Shift + Tab</kbd> - 上一個元素</li>
              <li><kbd>Enter/Space</kbd> - 啟動元素</li>
              <li><kbd>Escape</kbd> - 結束測試</li>
            </ul>
            <div class="focus-indicator" v-show="showFocusIndicator">
              目前焦點: {{ currentFocusElement }}
            </div>
          </div>
        </div>
      </div>

      <!-- 螢幕報讀器模擬 -->
      <div class="tool-card">
        <h3>🔊 螢幕報讀器模擬</h3>
        <div class="screen-reader-sim">
          <div class="sr-controls-header">
            <button @click="toggleScreenReaderMode" class="btn btn-primary">
              {{ screenReaderMode ? '停止模擬' : '啟動螢幕報讀器模擬' }}
            </button>
            <button @click="toggleSpeech" :class="['btn', speechEnabled ? 'btn-secondary' : 'btn-disabled']"
              :disabled="!screenReaderMode">
              {{ speechEnabled ? '🔊 語音開啟' : '🔇 語音關閉' }}
            </button>
          </div>

          <div v-if="screenReaderMode" class="sr-settings">
            <div class="setting-group">
              <label for="speech-rate">語音速度: {{ speechRate.toFixed(1) }}</label>
              <input id="speech-rate" type="range" min="0.5" max="2" step="0.1" v-model="speechRate"
                :disabled="!speechEnabled" />
            </div>
            <div class="setting-group">
              <label for="speech-volume">音量: {{ (speechVolume * 100).toFixed(0) }}%</label>
              <input id="speech-volume" type="range" min="0" max="1" step="0.1" v-model="speechVolume"
                :disabled="!speechEnabled" />
            </div>
            <div class="setting-group" v-if="availableVoices.length > 0">
              <label for="voice-select">語音選擇:</label>
              <select id="voice-select" v-model="selectedVoice" :disabled="!speechEnabled">
                <option v-for="voice in availableVoices" :key="voice.name" :value="voice">
                  {{ voice.name }} ({{ voice.lang }})
                </option>
              </select>
            </div>
          </div>

          <div v-if="screenReaderMode" class="sr-output">
            <h4>螢幕報讀器輸出:</h4>
            <div class="sr-text" aria-live="polite">{{ screenReaderOutput }}</div>
            <div class="sr-controls">
              <button @click="navigateElements('next')" class="btn btn-small">下一個元素</button>
              <button @click="navigateElements('prev')" class="btn btn-small">上一個元素</button>
              <button @click="readCurrentElement" class="btn btn-small">讀取當前元素</button>
              <button @click="stopSpeech" class="btn btn-small btn-warning" :disabled="!speechEnabled">停止語音</button>
            </div>
          </div>

          <div v-if="!speechEnabled && screenReaderMode" class="speech-warning">
            ⚠️ 此瀏覽器不支援語音合成功能，僅顯示文字輸出
          </div>
        </div>
      </div>

      <!-- 性能分析 -->
      <div class="tool-card">
        <h3>📊 無障礙性能分析</h3>
        <div class="performance-analysis">
          <button @click="analyzePerformance" class="btn btn-secondary">
            分析頁面性能
          </button>
          <div v-if="performanceData" class="performance-results">
            <div class="metric">
              <span class="metric-label">可互動元素數量:</span>
              <span class="metric-value">{{ performanceData.interactiveElements }}</span>
            </div>
            <div class="metric">
              <span class="metric-label">圖片 Alt 文字覆蓋率:</span>
              <span class="metric-value">{{ performanceData.altTextCoverage }}%</span>
            </div>
            <div class="metric">
              <span class="metric-label">標題結構完整性:</span>
              <span class="metric-value">{{ performanceData.headingStructure }}</span>
            </div>
            <div class="metric">
              <span class="metric-label">表單標籤關聯率:</span>
              <span class="metric-value">{{ performanceData.formLabelAssociation }}%</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue';

export default {
  name: 'A11yTestingTools',
  setup() {
    // Axe 測試相關
    const testing = ref(false);
    const axeResults = ref(null);

    // 色彩對比檢測
    const foregroundColor = ref('#000000');
    const backgroundColor = ref('#ffffff');

    // 鍵盤導航測試
    const keyboardTestActive = ref(false);
    const showFocusIndicator = ref(false);
    const currentFocusElement = ref('');

    // 螢幕報讀器模擬
    const screenReaderMode = ref(false);
    const screenReaderOutput = ref('');
    const currentElementIndex = ref(0);
    const speechEnabled = ref(false);
    const speechRate = ref(1);
    const speechVolume = ref(1);
    const speechPitch = ref(1);
    const availableVoices = ref([]);
    const selectedVoice = ref(null);

    // 性能分析
    const performanceData = ref(null);

    // 計算色彩對比度
    const hexToRgb = (hex) => {
      const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
      return result ? {
        r: parseInt(result[1], 16),
        g: parseInt(result[2], 16),
        b: parseInt(result[3], 16)
      } : null;
    };

    const getLuminance = (rgb) => {
      const { r, g, b } = rgb;
      const [rs, gs, bs] = [r, g, b].map(c => {
        c = c / 255;
        return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
      });
      return 0.2126 * rs + 0.7152 * gs + 0.0722 * bs;
    };

    const contrastRatio = computed(() => {
      const fg = hexToRgb(foregroundColor.value);
      const bg = hexToRgb(backgroundColor.value);

      if (!fg || !bg) return '1.00';

      const fgLum = getLuminance(fg);
      const bgLum = getLuminance(bg);

      const lighter = Math.max(fgLum, bgLum);
      const darker = Math.min(fgLum, bgLum);

      return ((lighter + 0.05) / (darker + 0.05)).toFixed(2);
    });

    const wcagAA = computed(() => ({
      normal: parseFloat(contrastRatio.value) >= 4.5,
      large: parseFloat(contrastRatio.value) >= 3.0
    }));

    const wcagAAA = computed(() => ({
      normal: parseFloat(contrastRatio.value) >= 7.0,
      large: parseFloat(contrastRatio.value) >= 4.5
    }));

    const contrastPreviewStyle = computed(() => ({
      color: foregroundColor.value,
      backgroundColor: backgroundColor.value,
      padding: '1rem',
      borderRadius: '4px',
      marginTop: '1rem',
      fontSize: '1.1rem'
    }));

    // Axe 測試函數
    const runAxeTest = async () => {
      testing.value = true;
      axeResults.value = null;

      try {
        // 模擬 axe-core 檢測 (實際使用時需要導入 axe-core)
        await new Promise(resolve => setTimeout(resolve, 2000));

        // 模擬檢測結果
        axeResults.value = {
          violations: [
            {
              help: '圖片必須有替代文字',
              description: '確保 img 元素有 alt 屬性',
              impact: 'critical',
              nodes: [
                { html: '<img src="example.jpg">' }
              ]
            },
            {
              help: '表單控制項必須有標籤',
              description: '確保每個表單元素都有對應的 label',
              impact: 'serious',
              nodes: [
                { html: '<input type="text" placeholder="姓名">' }
              ]
            }
          ],
          passes: [
            {
              help: '頁面必須有 main 地標',
              description: '文件必須有一個主要地標'
            },
            {
              help: '按鈕必須有可訪問的名稱',
              description: '確保按鈕有文字或 aria-label'
            }
          ],
          incomplete: [
            {
              help: '元素必須有足夠的色彩對比',
              description: '需要手動檢查背景圖片上的文字對比度'
            }
          ]
        };
      } catch (error) {
        console.error('Axe 測試失敗:', error);
      } finally {
        testing.value = false;
      }
    };

    // 鍵盤導航測試
    const startKeyboardTest = () => {
      keyboardTestActive.value = true;
      showFocusIndicator.value = true;

      const handleKeydown = (e) => {
        if (e.key === 'Tab') {
          setTimeout(() => {
            const activeElement = document.activeElement;
            currentFocusElement.value = getElementDescription(activeElement);
          }, 10);
        } else if (e.key === 'Escape') {
          endKeyboardTest();
        }
      };

      const handleFocusIn = (e) => {
        currentFocusElement.value = getElementDescription(e.target);
      };

      document.addEventListener('keydown', handleKeydown);
      document.addEventListener('focusin', handleFocusIn);

      // 清理函數
      const cleanup = () => {
        document.removeEventListener('keydown', handleKeydown);
        document.removeEventListener('focusin', handleFocusIn);
      };

      // 儲存清理函數供後續使用
      window.keyboardTestCleanup = cleanup;
    };

    const endKeyboardTest = () => {
      keyboardTestActive.value = false;
      showFocusIndicator.value = false;
      if (window.keyboardTestCleanup) {
        window.keyboardTestCleanup();
        delete window.keyboardTestCleanup;
      }
    };

    const getElementDescription = (element) => {
      if (!element) return '無元素';

      const tagName = element.tagName.toLowerCase();
      const text = element.textContent?.trim() ||
        element.getAttribute('aria-label') ||
        element.getAttribute('alt') ||
        element.getAttribute('title') ||
        element.getAttribute('placeholder') ||
        '未命名元素';

      return `${tagName}: ${text}`;
    };

    // 螢幕報讀器模擬
    const toggleScreenReaderMode = () => {
      screenReaderMode.value = !screenReaderMode.value;
      if (screenReaderMode.value) {
        const message = '螢幕報讀器模擬已啟動。使用控制按鈕來導航頁面元素。';
        screenReaderOutput.value = message;
        // 不自動撥放語音，僅顯示文字
        currentElementIndex.value = 0;
      } else {
        screenReaderOutput.value = '';
        stopSpeech();
      }
    };

    // 語音合成功能
    const initializeSpeech = () => {
      if ('speechSynthesis' in window) {
        speechEnabled.value = true;

        // 載入可用的語音
        const loadVoices = () => {
          const voices = speechSynthesis.getVoices();
          availableVoices.value = voices;

          // 優先選擇中文語音
          const chineseVoice = voices.find(voice =>
            voice.lang.includes('zh') || voice.lang.includes('cmn')
          );
          selectedVoice.value = chineseVoice || voices[0];
        };

        loadVoices();
        speechSynthesis.addEventListener('voiceschanged', loadVoices);
      } else {
        speechEnabled.value = false;
        console.warn('此瀏覽器不支援語音合成');
      }
    };

    const speakText = (text) => {
      if (!speechEnabled.value || !text) return;

      // 停止當前語音
      speechSynthesis.cancel();

      const utterance = new SpeechSynthesisUtterance(text);

      if (selectedVoice.value) {
        utterance.voice = selectedVoice.value;
      }

      utterance.rate = speechRate.value;
      utterance.volume = speechVolume.value;
      utterance.pitch = speechPitch.value;

      utterance.onstart = () => {
        console.log('開始朗讀:', text);
      };

      utterance.onend = () => {
        console.log('朗讀結束');
      };

      utterance.onerror = (event) => {
        console.error('語音合成錯誤:', event.error);
      };

      speechSynthesis.speak(utterance);
    };

    const stopSpeech = () => {
      if (speechEnabled.value) {
        speechSynthesis.cancel();
      }
    };

    const toggleSpeech = () => {
      speechEnabled.value = !speechEnabled.value;
      if (speechEnabled.value && screenReaderMode.value) {
        // 開啟語音時才撥放目前文字
        if (screenReaderOutput.value) {
          speakText(screenReaderOutput.value);
        }
      } else {
        stopSpeech();
      }
    };

    const getAllElements = () => {
      return Array.from(document.querySelectorAll('h1, h2, h3, h4, h5, h6, p, button, a, input, select, textarea, img, [role]'))
        .filter(el => el.offsetParent !== null); // 只包含可見元素
    };

    const navigateElements = (direction) => {
      const elements = getAllElements();
      if (elements.length === 0) return;

      if (direction === 'next') {
        currentElementIndex.value = (currentElementIndex.value + 1) % elements.length;
      } else {
        currentElementIndex.value = currentElementIndex.value === 0
          ? elements.length - 1
          : currentElementIndex.value - 1;
      }

      const element = elements[currentElementIndex.value];
      element.focus();
      readCurrentElement();
    };

    const readCurrentElement = () => {
      const elements = getAllElements();
      const element = elements[currentElementIndex.value];
      if (!element) return;

      let text = '';
      const tagName = element.tagName.toLowerCase();

      // 根據元素類型生成適當的朗讀文字
      switch (tagName) {
        case 'h1':
        case 'h2':
        case 'h3':
        case 'h4':
        case 'h5':
        case 'h6':
          text = `標題 ${tagName.charAt(1)} 級，${element.textContent}`;
          break;
        case 'button':
          text = `按鈕，${element.textContent || element.getAttribute('aria-label') || '未命名按鈕'}`;
          break;
        case 'a':
          text = `連結，${element.textContent || element.getAttribute('aria-label') || '未命名連結'}`;
          break;
        case 'input':
          const inputType = element.type || 'text';
          const label = element.getAttribute('aria-label') ||
            element.getAttribute('placeholder') ||
            '未命名輸入框';
          text = `${inputType} 輸入框，${label}`;
          break;
        case 'img':
          text = `圖片，${element.getAttribute('alt') || '無替代文字'}`;
          break;
        default:
          text = element.textContent || element.getAttribute('aria-label') || `${tagName} 元素`;
      }

      screenReaderOutput.value = text;

      // 如果啟用語音，則朗讀文字
      if (speechEnabled.value) {
        speakText(text);
      }
    };

    // 性能分析
    const analyzePerformance = () => {
      const interactiveElements = document.querySelectorAll('button, a, input, select, textarea, [tabindex]').length;

      const images = document.querySelectorAll('img');
      const imagesWithAlt = document.querySelectorAll('img[alt]');
      const altTextCoverage = images.length > 0 ? Math.round((imagesWithAlt.length / images.length) * 100) : 100;

      const headings = document.querySelectorAll('h1, h2, h3, h4, h5, h6');
      const hasH1 = document.querySelector('h1') !== null;
      const headingStructure = hasH1 && headings.length > 0 ? '良好' : '需改善';

      const formControls = document.querySelectorAll('input, select, textarea');
      const labeledControls = Array.from(formControls).filter(control => {
        return control.labels?.length > 0 ||
          control.getAttribute('aria-label') ||
          control.getAttribute('aria-labelledby');
      });
      const formLabelAssociation = formControls.length > 0
        ? Math.round((labeledControls.length / formControls.length) * 100)
        : 100;

      performanceData.value = {
        interactiveElements,
        altTextCoverage,
        headingStructure,
        formLabelAssociation
      };
    };

    onMounted(() => {
      initializeSpeech();
    });

    onUnmounted(() => {
      if (window.keyboardTestCleanup) {
        window.keyboardTestCleanup();
        delete window.keyboardTestCleanup;
      }
      stopSpeech();
    });

    return {
      // Axe 測試
      testing,
      axeResults,
      runAxeTest,

      // 色彩對比
      foregroundColor,
      backgroundColor,
      contrastRatio,
      wcagAA,
      wcagAAA,
      contrastPreviewStyle,

      // 鍵盤導航
      keyboardTestActive,
      showFocusIndicator,
      currentFocusElement,
      startKeyboardTest,

      // 螢幕報讀器
      screenReaderMode,
      screenReaderOutput,
      toggleScreenReaderMode,
      navigateElements,
      readCurrentElement,
      speechEnabled,
      speechRate,
      speechVolume,
      speechPitch,
      availableVoices,
      selectedVoice,
      speakText,
      stopSpeech,
      toggleSpeech,

      // 性能分析
      performanceData,
      analyzePerformance
    };
  }
};
</script>

<style scoped>
.a11y-testing-tools {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.tools-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 2rem;
  margin-top: 2rem;
}

.tool-card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 1.5rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.tool-card h3 {
  margin-top: 0;
  color: #1f2937;
  border-bottom: 2px solid #e5e7eb;
  padding-bottom: 0.5rem;
}

.btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 4px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  margin-right: 0.5rem;
  margin-bottom: 0.5rem;
}

.btn:focus {
  outline: 2px solid #3b82f6;
  outline-offset: 2px;
}

.btn-primary {
  background: #3b82f6;
  color: white;
}

.btn-primary:hover:not(:disabled) {
  background: #2563eb;
}

.btn-secondary {
  background: #6b7280;
  color: white;
}

.btn-secondary:hover:not(:disabled) {
  background: #4b5563;
}

.btn-small {
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.test-results {
  margin-top: 1rem;
  padding: 1rem;
  background: #f9fafb;
  border-radius: 4px;
  border: 1px solid #e5e7eb;
}

.result-summary {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  margin-bottom: 1rem;
}

.violations,
.passes,
.incomplete {
  padding: 0.25rem 0.75rem;
  border-radius: 4px;
  font-weight: 500;
  font-size: 0.875rem;
}

.violations {
  background: #fee2e2;
  color: #991b1b;
}

.passes {
  background: #dcfce7;
  color: #166534;
}

.incomplete {
  background: #fef3c7;
  color: #92400e;
}

.violations-list {
  margin-top: 1rem;
}

.violation-item {
  background: white;
  border: 1px solid #fecaca;
  border-radius: 4px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.violation-item strong {
  color: #dc2626;
}

.violation-item details {
  margin-top: 0.5rem;
}

.violation-item pre {
  background: #f3f4f6;
  padding: 0.5rem;
  border-radius: 4px;
  font-size: 0.875rem;
  overflow-x: auto;
}

.color-contrast-tool {
  display: grid;
  gap: 1rem;
}

.color-input-group {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.color-input-group label {
  min-width: 80px;
  font-weight: 500;
}

.color-input-group input[type="color"] {
  width: 50px;
  height: 40px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  cursor: pointer;
}

.color-input-group input[type="text"] {
  padding: 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-family: monospace;
}

.contrast-results {
  background: #f9fafb;
  padding: 1rem;
  border-radius: 4px;
}

.contrast-ratio {
  font-size: 1.25rem;
  margin-bottom: 1rem;
}

.compliance-item {
  padding: 0.5rem;
  margin: 0.25rem 0;
  border-radius: 4px;
  font-weight: 500;
}

.compliance-item.pass {
  background: #dcfce7;
  color: #166534;
}

.compliance-item.fail {
  background: #fee2e2;
  color: #991b1b;
}

.keyboard-instructions {
  margin-top: 1rem;
  padding: 1rem;
  background: #f0f9ff;
  border-radius: 4px;
}

.keyboard-instructions ul {
  margin: 1rem 0;
  padding-left: 1.5rem;
}

.keyboard-instructions kbd {
  background: #374151;
  color: white;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-family: monospace;
  font-size: 0.875rem;
}

.focus-indicator {
  background: #dbeafe;
  border: 2px solid #3b82f6;
  padding: 0.75rem;
  border-radius: 4px;
  font-weight: 500;
  margin-top: 1rem;
}

.screen-reader-sim {
  display: grid;
  gap: 1rem;
}

.sr-controls-header {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  align-items: center;
}

.sr-settings {
  background: #f8fafc;
  padding: 1rem;
  border-radius: 4px;
  border: 1px solid #e2e8f0;
  display: grid;
  gap: 1rem;
}

.setting-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.setting-group label {
  font-weight: 500;
  color: #374151;
  font-size: 0.875rem;
}

.setting-group input[type="range"] {
  width: 100%;
  height: 6px;
  border-radius: 3px;
  background: #d1d5db;
  outline: none;
  transition: background 0.3s;
}

.setting-group input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  transition: background 0.3s;
}

.setting-group input[type="range"]::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  border: none;
}

.setting-group input[type="range"]:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.setting-group select {
  padding: 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  background: white;
  font-size: 0.875rem;
}

.setting-group select:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.btn-disabled {
  background: #9ca3af;
  color: white;
}

.btn-disabled:hover {
  background: #9ca3af;
}

.btn-warning {
  background: #f59e0b;
  color: white;
}

.btn-warning:hover:not(:disabled) {
  background: #d97706;
}

.speech-warning {
  background: #fef3c7;
  border: 1px solid #f59e0b;
  color: #92400e;
  padding: 1rem;
  border-radius: 4px;
  font-weight: 500;
  text-align: center;
}

.sr-output {
  background: #1f2937;
  color: #f9fafb;
  padding: 1rem;
  border-radius: 4px;
}

.sr-text {
  background: #374151;
  padding: 1rem;
  border-radius: 4px;
  min-height: 3rem;
  font-family: monospace;
  margin-bottom: 1rem;
}

.sr-controls {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.performance-analysis {
  display: grid;
  gap: 1rem;
}

.performance-results {
  background: #f9fafb;
  padding: 1rem;
  border-radius: 4px;
}

.metric {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem 0;
  border-bottom: 1px solid #e5e7eb;
}

.metric:last-child {
  border-bottom: none;
}

.metric-label {
  font-weight: 500;
}

.metric-value {
  font-weight: 600;
  color: #3b82f6;
}

@media (max-width: 768px) {
  .tools-grid {
    grid-template-columns: 1fr;
  }

  .color-input-group {
    flex-direction: column;
    align-items: flex-start;
  }

  .result-summary {
    flex-direction: column;
  }
}
</style>
