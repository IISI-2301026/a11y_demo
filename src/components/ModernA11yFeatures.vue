<template>
  <div class="modern-features">
    <h2>🎯 現代無障礙技術演示</h2>

    <!-- Web Speech API -->
    <section class="demo-section">
      <h3>🗣️ 語音朗讀 API (Web Speech API)</h3>
      <div class="speech-demo">
        <textarea v-model="speechText" placeholder="輸入要朗讀的文字..." rows="3" aria-label="輸入要朗讀的文字"></textarea>
        <div class="speech-controls">
          <button @click="speak" :disabled="isSpeaking" class="btn btn-primary">
            {{ isSpeaking ? '朗讀中...' : '開始朗讀' }}
          </button>
          <button @click="stopSpeaking" :disabled="!isSpeaking" class="btn btn-secondary">
            停止朗讀
          </button>
          <button @click="testVoice" class="btn btn-info">
            🎤 測試語音
          </button>
        </div>

        <div class="speech-settings">
          <div class="setting-group">
            <label for="voice-select">語音選擇:</label>
            <select id="voice-select" v-model="selectedVoice" @change="updateVoice">
              <option v-for="(voice, index) in availableVoices" :key="index" :value="index">
                {{ voice.name }} ({{ voice.lang }})
              </option>
            </select>
          </div>

          <div class="setting-group">
            <label for="speech-rate">語速: {{ speechRate }}x</label>
            <input id="speech-rate" type="range" v-model="speechRate" min="0.5" max="2" step="0.1" />
          </div>

          <div class="setting-group">
            <label for="speech-pitch">音調: {{ speechPitch }}</label>
            <input id="speech-pitch" type="range" v-model="speechPitch" min="0" max="2" step="0.1" />
          </div>

          <div class="setting-group">
            <label for="speech-volume">音量: {{ Math.round(speechVolume * 100) }}%</label>
            <input id="speech-volume" type="range" v-model="speechVolume" min="0" max="1" step="0.1" />
          </div>
        </div>

        <div class="speech-status">
          <div v-if="speechSupported" class="status-good">
            ✅ 您的瀏覽器支援語音合成功能
          </div>
          <div v-else class="status-error">
            ❌ 您的瀏覽器不支援語音合成功能
          </div>
          <div v-if="availableVoices.length > 0" class="voice-count">
            📢 可用語音: {{ availableVoices.length }} 種
          </div>
          <div v-if="isSpeaking" class="status-speaking">
            🎤 正在朗讀中...
          </div>
        </div>
      </div>
    </section>

    <!-- Intersection Observer API -->
    <section class="demo-section">
      <h3>👁️ 視覺化進度追蹤 (Intersection Observer)</h3>
      <div class="progress-tracking">
        <div class="progress-explanation">
          <p><strong>功能說明：</strong>這個功能會即時追蹤您正在閱讀的內容區塊，當您滾動下方的內容區域時：</p>
          <ul>
            <li>📊 上方進度條會顯示總體閱讀進度</li>
            <li>💙 正在閱讀的段落會以藍色高亮顯示</li>
            <li>🔍 這是 Intersection Observer API 的實際應用</li>
          </ul>
        </div>

        <div class="progress-display">
          <div class="progress-bar">
            <div class="progress-fill" :style="{ width: readingProgress + '%' }"></div>
          </div>
          <div class="progress-stats">
            <span>總進度: <strong>{{ Math.round(readingProgress) }}%</strong></span>
            <span>正在閱讀: <strong>{{ sectionsInView.length }}</strong> / {{ sections.length }} 段</span>
            <span v-if="sectionsInView.length > 0">當前段落: <strong>第 {{ Math.min(...sectionsInView) + 1 }}
                段</strong></span>
          </div>
        </div>

        <div ref="scrollContent" class="scroll-content">
          <div class="scroll-instruction">
            <p>📖 <strong>試試看：</strong>在下方內容區域中滾動，觀察進度條和段落高亮的變化！</p>
          </div>
          <div v-for="(section, index) in sections" :key="index" :ref="el => sectionRefs[index] = el"
            class="scroll-section" :class="{
              'in-view': sectionsInView.includes(index),
              'fully-read': index < Math.min(...sectionsInView.length > 0 ? sectionsInView : [0])
            }">
            <h4>📄 第 {{ index + 1 }} 段內容
              <span v-if="sectionsInView.includes(index)" class="reading-indicator">👁️ 正在閱讀</span>
            </h4>
            <p>{{ section.content }}</p>
            <div class="section-footer">
              <small>段落 {{ index + 1 }} / {{ sections.length }}</small>
            </div>
          </div>

          <div class="completion-message" v-if="readingProgress >= 90">
            <div class="celebration">
              🎉 <strong>恭喜！</strong>您已經閱讀了大部分內容！
            </div>
          </div>
        </div>

        <div class="technical-info">
          <details>
            <summary>🔧 技術詳情：Intersection Observer API</summary>
            <div class="tech-details">
              <p>這個功能使用了現代瀏覽器的 <code>Intersection Observer API</code>：</p>
              <ul>
                <li><strong>高效能：</strong>不需要監聽 scroll 事件</li>
                <li><strong>精確追蹤：</strong>可以偵測元素進入/離開視窗</li>
                <li><strong>可設定閾值：</strong>目前設定為 50% 可見時觸發</li>
                <li><strong>無障礙應用：</strong>可用於通知螢幕報讀器用戶的閱讀進度</li>
              </ul>
              <p><strong>偵測到的段落：</strong> {{ sectionsInView.join(', ') || '無' }}</p>
            </div>
          </details>
        </div>
      </div>
    </section>

    <!-- Media Query 偵測 -->
    <section class="demo-section">
      <h3>📱 響應式偏好設定偵測</h3>
      <div class="preference-detection">
        <div class="preference-card">
          <h4>🔍 系統偏好設定即時偵測</h4>
          <div class="preference-explanation">
            <p><strong>這個功能會即時監測您的系統偏好設定變化：</strong></p>
            <ul>
              <li>🌙 <strong>深色模式：</strong>偵測系統是否啟用深色主題</li>
              <li>🎭 <strong>減少動畫：</strong>偵測是否開啟「減少動畫」無障礙選項</li>
              <li>🔆 <strong>高對比：</strong>偵測是否啟用高對比顯示模式</li>
            </ul>
          </div>

          <div class="current-preferences">
            <h5>目前偵測到的設定：</h5>
            <div class="preference-list">
              <div class="preference-item">
                <span class="preference-label">深色模式:</span>
                <span :class="['preference-value', darkModePreference === '啟用' ? 'enabled' : 'disabled']">
                  {{ darkModePreference }}
                </span>
              </div>
              <div class="preference-item">
                <span class="preference-label">減少動畫:</span>
                <span :class="['preference-value', reducedMotionPreference === '啟用' ? 'enabled' : 'disabled']">
                  {{ reducedMotionPreference }}
                </span>
              </div>
              <div class="preference-item">
                <span class="preference-label">高對比:</span>
                <span :class="['preference-value', highContrastPreference === '啟用' ? 'enabled' : 'disabled']">
                  {{ highContrastPreference }}
                </span>
              </div>
            </div>
          </div>

          <div class="testing-instructions">
            <h5>🧪 如何測試偏好設定偵測：</h5>
            <div class="test-steps">
              <div class="test-step">
                <strong>Windows 11：</strong>
                <ul>
                  <li>深色模式：設定 → 個人化 → 色彩 → 選擇「深色」</li>
                  <li>減少動畫：設定 → 協助工具 → 視覺效果 → 關閉「動畫效果」</li>
                  <li>高對比：設定 → 協助工具 → 高對比 → 開啟高對比模式</li>
                </ul>
              </div>
              <!-- <div class="test-step">
                <strong>macOS：</strong>
                <ul>
                  <li>深色模式：系統偏好設定 → 一般 → 外觀</li>
                  <li>減少動畫：系統偏好設定 → 輔助使用 → 顯示 → 減少動態效果</li>
                  <li>高對比：系統偏好設定 → 輔助使用 → 顯示 → 增加對比</li>
                </ul>
              </div> -->
              <div class="test-step">
                <strong>瀏覽器開發者工具：</strong>
                <ul>
                  <li>打開開發者工具 (F12)</li>
                  <li>按下 Ctrl+Shift+P (或 Cmd+Shift+P)</li>
                  <li>搜尋「Emulate CSS prefers-color-scheme」</li>
                  <li>選擇不同的選項來模擬偏好設定</li>
                </ul>
              </div>
            </div>
          </div>
        </div>

        <div class="preference-actions">
          <button @click="applyPreferences" class="btn btn-primary">
            🎨 套用系統偏好設定
          </button>
          <button @click="resetPreferences" class="btn btn-secondary">
            🔄 重置偏好設定
          </button>
          <button @click="detectPreferences" class="btn btn-info">
            🔍 重新偵測
          </button>
        </div>
      </div>
    </section>

    <!-- Focus Management -->
    <section class="demo-section">
      <h3>🎯 進階焦點管理</h3>
      <div class="focus-management">
        <button @click="openModal" class="btn btn-primary">開啟模態視窗</button>

        <!-- 模態視窗 -->
        <div v-if="modalOpen" class="modal-overlay" @click="closeModal">
          <div class="modal-content" @click.stop ref="modalContent" role="dialog" aria-labelledby="modal-title"
            aria-modal="true">
            <h4 id="modal-title">焦點陷阱演示</h4>
            <p>在這個模態視窗中，Tab 鍵會在視窗內的元素間循環，不會跳到背景內容。</p>
            <input type="text" placeholder="輸入框 1" />
            <input type="text" placeholder="輸入框 2" />
            <div class="modal-actions">
              <button @click="closeModal" class="btn btn-secondary">取消</button>
              <button @click="closeModal" class="btn btn-primary">確認</button>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- 無障礙 Vue 3 Composition API 示例 -->
    <section class="demo-section">
      <h3>⚡ 系統偏好設定偵測 無障礙 Hook</h3>
      <div class="composition-demo">
        <div class="announcement-region" aria-live="polite" aria-atomic="true">
          {{ announcement }}
        </div>
        <button @click="makeAnnouncement" class="btn btn-primary">
          觸發無障礙通知
        </button>
        <button @click="focusFirstInput" class="btn btn-secondary">
          焦點移至第一個輸入框
        </button>
        <input ref="firstInput" type="text" placeholder="第一個輸入框" />
        <input type="text" placeholder="第二個輸入框" />
      </div>
    </section>
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, nextTick, computed } from 'vue';

export default {
  name: 'ModernA11yFeatures',
  setup() {
    // 語音朗讀相關
    const speechText = ref('歡迎使用現代無障礙技術演示！這個功能展示了如何使用 Web Speech API 來增強網站的可訪問性。您現在聽到的是瀏覽器內建的語音合成功能。');
    const isSpeaking = ref(false);
    const speechRate = ref(1);
    const speechPitch = ref(1);
    const speechVolume = ref(0.8);
    const selectedVoice = ref(0);
    const availableVoices = ref([]);
    const speechSupported = ref(false);

    // 視覺化進度追蹤
    const readingProgress = ref(0);
    const sectionsInView = ref([]);
    const sectionRefs = ref([]);
    const sections = ref([
      { content: '這是第一段示例內容，展示了如何使用 Intersection Observer API 來追蹤用戶的閱讀進度。' },
      { content: '第二段內容介紹了現代瀏覽器的無障礙功能，包括語音合成和螢幕報讀器支援。' },
      { content: '第三段內容討論了響應式設計中的無障礙考量，如何適應不同裝置和用戶需求。' },
      { content: '第四段內容展示了進階的焦點管理技術，確保鍵盤用戶能夠順暢地導航。' },
      { content: '最後一段總結了 Vue 3 Composition API 在無障礙開發中的優勢和最佳實踐。' }
    ]);

    // 偏好設定
    const darkModePreference = ref('未知');
    const reducedMotionPreference = ref('未知');
    const highContrastPreference = ref('未知');

    // 模態視窗
    const modalOpen = ref(false);
    const modalContent = ref(null);
    const lastFocusedElement = ref(null);

    // 通知
    const announcement = ref('');
    const firstInput = ref(null);

    // 語音朗讀功能
    const currentUtterance = ref(null);
    const isStoppedByUser = ref(false);

    const speak = () => {
      if ('speechSynthesis' in window) {
        // 如果正在朗讀，先停止
        if (isSpeaking.value) {
          stopSpeaking();
        }

        isStoppedByUser.value = false;
        const utterance = new SpeechSynthesisUtterance(speechText.value);
        currentUtterance.value = utterance;

        utterance.rate = speechRate.value;
        utterance.pitch = speechPitch.value;
        utterance.volume = speechVolume.value;

        // 設定語音
        if (availableVoices.value.length > 0) {
          utterance.voice = availableVoices.value[selectedVoice.value];
        }

        utterance.onstart = () => {
          isSpeaking.value = true;
          console.log('🎤 開始語音朗讀');
        };

        utterance.onend = () => {
          if (!isStoppedByUser.value) {
            isSpeaking.value = false;
            console.log('🎤 語音朗讀自然結束');
          }
          currentUtterance.value = null;
        };

        utterance.onerror = (event) => {
          // 只有在不是用戶主動停止時才顯示錯誤
          if (!isStoppedByUser.value) {
            console.error('🎤 語音朗讀錯誤:', event.error);
            // 只有在真正的錯誤時才顯示 alert（不包括被中斷的情況）
            if (event.error !== 'interrupted' && event.error !== 'canceled') {
              alert(`語音朗讀發生錯誤: ${event.error}`);
            }
          }
          isSpeaking.value = false;
          currentUtterance.value = null;
        };

        speechSynthesis.speak(utterance);
      } else {
        alert('您的瀏覽器不支援語音合成功能');
      }
    };

    const stopSpeaking = () => {
      if ('speechSynthesis' in window && isSpeaking.value) {
        isStoppedByUser.value = true;
        speechSynthesis.cancel();
        isSpeaking.value = false;
        currentUtterance.value = null;
        console.log('🎤 語音朗讀已由用戶停止');
      }
    };

    const testVoice = () => {
      const originalText = speechText.value;
      speechText.value = '這是語音測試，您可以聽到這段話來確認語音功能正常運作。';
      speak();
      setTimeout(() => {
        speechText.value = originalText;
      }, 100);
    };

    const loadVoices = () => {
      if ('speechSynthesis' in window) {
        speechSupported.value = true;
        const voices = speechSynthesis.getVoices();
        availableVoices.value = voices;

        // 優先選擇中文語音
        const chineseVoiceIndex = voices.findIndex(voice =>
          voice.lang.includes('zh') || voice.lang.includes('ch')
        );
        if (chineseVoiceIndex !== -1) {
          selectedVoice.value = chineseVoiceIndex;
        }

        console.log('🎤 可用語音數量:', voices.length);
        console.log('🎤 預設語音:', voices[selectedVoice.value]?.name);
      } else {
        speechSupported.value = false;
      }
    };

    const updateVoice = () => {
      console.log('🎤 語音已切換至:', availableVoices.value[selectedVoice.value]?.name);
    };

    // Intersection Observer
    let observer = null;

    const setupIntersectionObserver = () => {
      if (!window.IntersectionObserver) return;

      observer = new IntersectionObserver((entries) => {
        const inView = [];
        entries.forEach(entry => {
          const index = sectionRefs.value.indexOf(entry.target);
          if (entry.isIntersecting && index !== -1) {
            inView.push(index);
          }
        });

        sectionsInView.value = inView.sort((a, b) => a - b);

        // 改進進度計算：根據最高可見段落計算
        if (sectionsInView.value.length > 0) {
          const maxVisibleSection = Math.max(...sectionsInView.value);
          // 進度 = (最高可見段落 + 1) / 總段落數 * 100
          readingProgress.value = ((maxVisibleSection + 1) / sections.value.length) * 100;
        } else {
          readingProgress.value = 0;
        }

        // 無障礙通知：當進度有顯著變化時通知螢幕報讀器
        const currentProgress = Math.round(readingProgress.value);
        if (currentProgress > 0 && currentProgress % 25 === 0) {
          const prevProgress = Math.round(((Math.max(...sectionsInView.value) || 0) / sections.value.length) * 100);
          if (Math.abs(currentProgress - prevProgress) >= 25) {
            announcement.value = `閱讀進度已達到 ${currentProgress}%`;
            setTimeout(() => {
              announcement.value = '';
            }, 2000);
          }
        }
      }, {
        threshold: [0.1, 0.5, 0.9], // 多個閾值，更精確的偵測
        rootMargin: '-50px 0px' // 減少邊緣誤觸
      });

      nextTick(() => {
        sectionRefs.value.forEach(el => {
          if (el) observer.observe(el);
        });
      });
    };

    // 系統偏好設定偵測
    const mediaQueries = ref({});

    const detectPreferences = () => {
      if (window.matchMedia) {
        // 深色模式
        const darkMode = window.matchMedia('(prefers-color-scheme: dark)');
        darkModePreference.value = darkMode.matches ? '啟用' : '停用';
        mediaQueries.value.darkMode = darkMode;

        // 減少動畫
        const reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)');
        reducedMotionPreference.value = reducedMotion.matches ? '啟用' : '停用';
        mediaQueries.value.reducedMotion = reducedMotion;

        // 高對比
        const highContrast = window.matchMedia('(prefers-contrast: high)');
        highContrastPreference.value = highContrast.matches ? '啟用' : '停用';
        mediaQueries.value.highContrast = highContrast;

        // 設定即時監聽器
        setupPreferenceListeners();
      }
    };

    const setupPreferenceListeners = () => {
      // 深色模式變化監聽
      if (mediaQueries.value.darkMode) {
        mediaQueries.value.darkMode.addEventListener('change', (e) => {
          darkModePreference.value = e.matches ? '啟用' : '停用';
          announcement.value = `系統深色模式已${e.matches ? '啟用' : '停用'}`;
          console.log('🌙 深色模式偏好變更:', e.matches);
        });
      }

      // 減少動畫變化監聽
      if (mediaQueries.value.reducedMotion) {
        mediaQueries.value.reducedMotion.addEventListener('change', (e) => {
          reducedMotionPreference.value = e.matches ? '啟用' : '停用';
          announcement.value = `減少動畫偏好已${e.matches ? '啟用' : '停用'}`;
          console.log('🎭 減少動畫偏好變更:', e.matches);
        });
      }

      // 高對比變化監聽
      if (mediaQueries.value.highContrast) {
        mediaQueries.value.highContrast.addEventListener('change', (e) => {
          highContrastPreference.value = e.matches ? '啟用' : '停用';
          announcement.value = `高對比偏好已${e.matches ? '啟用' : '停用'}`;
          console.log('🔆 高對比偏好變更:', e.matches);
        });
      }
    };

    const applyPreferences = () => {
      const body = document.body;

      // 清除現有的偏好設定 class
      body.classList.remove('dark-mode', 'reduced-motion', 'high-contrast');

      if (darkModePreference.value === '啟用') {
        body.classList.add('dark-mode');
      }

      if (reducedMotionPreference.value === '啟用') {
        body.classList.add('reduced-motion');
      }

      if (highContrastPreference.value === '啟用') {
        body.classList.add('high-contrast');
      }

      announcement.value = '已套用系統偏好設定到頁面樣式';

      // 記錄應用的偏好設定
      const appliedPrefs = [];
      if (darkModePreference.value === '啟用') appliedPrefs.push('深色模式');
      if (reducedMotionPreference.value === '啟用') appliedPrefs.push('減少動畫');
      if (highContrastPreference.value === '啟用') appliedPrefs.push('高對比');

      console.log('🎨 已套用偏好設定:', appliedPrefs.length > 0 ? appliedPrefs.join(', ') : '無特殊偏好');
    };

    const resetPreferences = () => {
      const body = document.body;
      body.classList.remove('dark-mode', 'reduced-motion', 'high-contrast');
      announcement.value = '已重置所有偏好設定樣式';
      console.log('🔄 已重置偏好設定樣式');
    };

    // 模態視窗焦點管理
    const openModal = () => {
      lastFocusedElement.value = document.activeElement;
      modalOpen.value = true;

      nextTick(() => {
        if (modalContent.value) {
          modalContent.value.focus();
          setupFocusTrap();
        }
      });
    };

    const closeModal = () => {
      modalOpen.value = false;
      if (lastFocusedElement.value) {
        lastFocusedElement.value.focus();
      }
    };

    const setupFocusTrap = () => {
      const focusableElements = modalContent.value.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );

      const firstElement = focusableElements[0];
      const lastElement = focusableElements[focusableElements.length - 1];

      const handleTabKey = (e) => {
        if (e.key === 'Tab') {
          if (e.shiftKey) {
            if (document.activeElement === firstElement) {
              lastElement.focus();
              e.preventDefault();
            }
          } else {
            if (document.activeElement === lastElement) {
              firstElement.focus();
              e.preventDefault();
            }
          }
        }

        if (e.key === 'Escape') {
          closeModal();
        }
      };

      document.addEventListener('keydown', handleTabKey);

      // 清理函數會在組件卸載時自動調用
      onUnmounted(() => {
        document.removeEventListener('keydown', handleTabKey);
      });
    };

    // 無障礙通知和焦點管理
    const makeAnnouncement = () => {
      const messages = [
        '這是一個無障礙通知示例',
        '內容已更新',
        '操作已完成',
        '狀態已變更'
      ];
      announcement.value = messages[Math.floor(Math.random() * messages.length)];

      setTimeout(() => {
        announcement.value = '';
      }, 3000);
    };

    const focusFirstInput = () => {
      if (firstInput.value) {
        firstInput.value.focus();
      }
    };

    onMounted(() => {
      setupIntersectionObserver();
      detectPreferences();

      // 載入語音選項
      loadVoices();

      // 監聽語音載入完成事件
      if ('speechSynthesis' in window) {
        speechSynthesis.onvoiceschanged = loadVoices;
      }
    });

    onUnmounted(() => {
      if (observer) {
        observer.disconnect();
      }
      stopSpeaking();
    });

    return {
      // 語音朗讀
      speechText,
      isSpeaking,
      speechRate,
      speechPitch,
      speechVolume,
      selectedVoice,
      availableVoices,
      speechSupported,
      speak,
      stopSpeaking,
      testVoice,
      updateVoice,

      // 進度追蹤
      readingProgress,
      sectionsInView,
      sectionRefs,
      sections,

      // 偏好設定
      darkModePreference,
      reducedMotionPreference,
      highContrastPreference,
      applyPreferences,
      resetPreferences,
      detectPreferences,

      // 模態視窗
      modalOpen,
      modalContent,
      openModal,
      closeModal,

      // 通知和焦點
      announcement,
      firstInput,
      makeAnnouncement,
      focusFirstInput
    };
  }
};
</script>

<style scoped>
.modern-features {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
}

.demo-section {
  margin-bottom: 3rem;
  padding: 2rem;
  border-radius: 8px;
  background: #f8fafc;
  border: 1px solid #e2e8f0;
}

.speech-demo textarea {
  width: 100%;
  margin-bottom: 1rem;
  padding: 0.75rem;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-family: inherit;
}

.speech-controls {
  display: flex;
  gap: 1rem;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 1rem;
}

.speech-settings {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin: 1rem 0;
  padding: 1rem;
  background: #f8fafc;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
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

.setting-group select {
  padding: 0.5rem;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  background: white;
  font-size: 0.875rem;
}

.setting-group input[type="range"] {
  width: 100%;
  height: 6px;
  border-radius: 3px;
  background: #e5e7eb;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
}

.setting-group input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  border: 2px solid white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.setting-group input[type="range"]::-moz-range-thumb {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  border: 2px solid white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.speech-status {
  margin-top: 1rem;
  padding: 1rem;
  border-radius: 6px;
  background: #f9fafb;
}

.status-good {
  color: #059669;
  font-weight: 500;
  margin-bottom: 0.5rem;
}

.status-error {
  color: #dc2626;
  font-weight: 500;
  margin-bottom: 0.5rem;
}

.status-speaking {
  color: #0ea5e9;
  font-weight: 500;
  margin-bottom: 0.5rem;
  animation: pulse 2s infinite;
}

@keyframes pulse {

  0%,
  100% {
    opacity: 1;
  }

  50% {
    opacity: 0.5;
  }
}

.voice-count {
  color: #6b7280;
  font-size: 0.875rem;
}

.btn-info {
  background: #0ea5e9;
  color: white;
}

.btn-info:hover:not(:disabled) {
  background: #0284c7;
}

.progress-tracking {
  margin-top: 1rem;
}

.progress-explanation {
  background: #f0f9ff;
  border: 1px solid #0ea5e9;
  border-radius: 6px;
  padding: 1rem;
  margin-bottom: 1rem;
}

.progress-explanation ul {
  margin: 0.5rem 0 0 1.5rem;
  padding: 0;
}

.progress-explanation li {
  margin: 0.25rem 0;
}

.progress-display {
  margin-bottom: 1rem;
}

.progress-bar {
  width: 100%;
  height: 12px;
  background: #e5e7eb;
  border-radius: 6px;
  overflow: hidden;
  margin-bottom: 0.75rem;
  position: relative;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #3b82f6, #06b6d4, #10b981);
  transition: width 0.5s ease;
  position: relative;
}

.progress-fill::after {
  content: '';
  position: absolute;
  top: 0;
  right: -10px;
  width: 20px;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.4), transparent);
  animation: shimmer 2s infinite;
}

@keyframes shimmer {
  0% {
    transform: translateX(-20px);
  }

  100% {
    transform: translateX(20px);
  }
}

.progress-stats {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  font-size: 0.875rem;
  color: #6b7280;
}

.progress-stats span {
  background: white;
  padding: 0.25rem 0.75rem;
  border-radius: 4px;
  border: 1px solid #e5e7eb;
}

.scroll-instruction {
  background: #fef3c7;
  border: 1px solid #f59e0b;
  border-radius: 4px;
  padding: 0.75rem;
  margin-bottom: 1rem;
  text-align: center;
}

.scroll-content {
  max-height: 400px;
  overflow-y: auto;
  border: 2px solid #d1d5db;
  border-radius: 8px;
  padding: 1rem;
  background: #fafafa;
}

.scroll-section {
  padding: 1.5rem;
  margin-bottom: 1rem;
  border-radius: 6px;
  transition: all 0.4s ease;
  border: 2px solid transparent;
  background: white;
  position: relative;
}

.scroll-section.in-view {
  background: #dbeafe;
  border-color: #3b82f6;
  box-shadow: 0 4px 6px -1px rgba(59, 130, 246, 0.1);
  transform: scale(1.02);
}

.scroll-section.fully-read {
  background: #d1fae5;
  border-color: #10b981;
}

.reading-indicator {
  display: inline-block;
  background: #3b82f6;
  color: white;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  margin-left: 0.5rem;
  animation: blink 1.5s infinite;
}

@keyframes blink {

  0%,
  50% {
    opacity: 1;
  }

  51%,
  100% {
    opacity: 0.5;
  }
}

.section-footer {
  border-top: 1px solid #e5e7eb;
  padding-top: 0.5rem;
  margin-top: 1rem;
  text-align: right;
  color: #9ca3af;
}

.completion-message {
  background: #dcfce7;
  border: 2px solid #16a34a;
  border-radius: 8px;
  padding: 1rem;
  margin-top: 1rem;
  text-align: center;
}

.celebration {
  color: #16a34a;
  font-size: 1.1rem;
  animation: celebration 2s ease-in-out infinite;
}

@keyframes celebration {

  0%,
  100% {
    transform: scale(1);
  }

  50% {
    transform: scale(1.05);
  }
}

.technical-info {
  margin-top: 1rem;
  background: #f9fafb;
  border-radius: 6px;
  overflow: hidden;
}

.technical-info summary {
  padding: 1rem;
  background: #f3f4f6;
  cursor: pointer;
  font-weight: 500;
  user-select: none;
}

.technical-info summary:hover {
  background: #e5e7eb;
}

.tech-details {
  padding: 1rem;
}

.tech-details code {
  background: #f3f4f6;
  padding: 0.25rem 0.5rem;
  border-radius: 3px;
  font-family: monospace;
  font-size: 0.875rem;
}

.tech-details ul {
  margin: 0.5rem 0 0 1.5rem;
}

.tech-details li {
  margin: 0.5rem 0;
}

.preference-detection {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.preference-card {
  padding: 2rem;
  background: white;
  border-radius: 12px;
  border: 1px solid #d1d5db;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
}

.preference-explanation {
  background: #f0f9ff;
  border: 1px solid #0ea5e9;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.preference-explanation ul {
  margin: 0.5rem 0 0 1.5rem;
  padding: 0;
}

.preference-explanation li {
  margin: 0.5rem 0;
}

.current-preferences {
  margin-bottom: 1.5rem;
}

.current-preferences h5 {
  color: #374151;
  margin-bottom: 1rem;
  font-size: 1.1rem;
}

.preference-list {
  background: #f9fafb;
  border-radius: 8px;
  padding: 1rem;
  border: 1px solid #e5e7eb;
}

.preference-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.75rem 0;
  border-bottom: 1px solid #e5e7eb;
}

.preference-item:last-child {
  border-bottom: none;
}

.preference-label {
  font-weight: 500;
  color: #374151;
}

.preference-value {
  padding: 0.25rem 0.75rem;
  border-radius: 4px;
  font-weight: 600;
  font-size: 0.875rem;
}

.preference-value.enabled {
  background: #dcfce7;
  color: #16a34a;
  border: 1px solid #bbf7d0;
}

.preference-value.disabled {
  background: #fef2f2;
  color: #dc2626;
  border: 1px solid #fecaca;
}

.testing-instructions {
  background: #fffbeb;
  border: 1px solid #f59e0b;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 1.5rem;
}

.testing-instructions h5 {
  color: #92400e;
  margin-bottom: 1rem;
}

.test-steps {
  display: grid;
  gap: 1rem;
}

.test-step {
  background: white;
  padding: 1rem;
  border-radius: 6px;
  border: 1px solid #fde68a;
}

.test-step strong {
  color: #92400e;
  display: block;
  margin-bottom: 0.5rem;
}

.test-step ul {
  margin: 0.5rem 0 0 1.5rem;
  padding: 0;
}

.test-step li {
  margin: 0.25rem 0;
  font-size: 0.9rem;
}

.preference-actions {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
  justify-content: center;
}

/* 舊的樣式保留以防相容性問題 */
.preference-card ul {
  list-style: none;
  padding: 0;
  margin: 1rem 0 0 0;
}

.preference-card li {
  padding: 0.5rem 0;
  display: flex;
  justify-content: space-between;
}

.preference-card span.啟用 {
  color: #16a34a;
  font-weight: bold;
}

.preference-card span.停用 {
  color: #dc2626;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
}

.modal-content:focus {
  outline: 2px solid #3b82f6;
  outline-offset: -2px;
}

.modal-actions {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  margin-top: 1.5rem;
}

.composition-demo {
  display: grid;
  gap: 1rem;
}

.announcement-region {
  min-height: 2rem;
  padding: 1rem;
  background: #fef3c7;
  border: 1px solid #f59e0b;
  border-radius: 4px;
  font-weight: bold;
}

.btn {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 4px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  text-decoration: none;
  display: inline-block;
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

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

.dark-mode {
  background: #1f2937;
  color: #f9fafb;
}

.high-contrast {
  filter: contrast(150%);
}
</style>
