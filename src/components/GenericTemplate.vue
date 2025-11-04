<template>
  <div class="generic-template" :class="{ 'is-desktop': isDesktop }">
    <!-- Toolbar -->
    <div class="toolbar" :class="{ 'toolbar-desktop': isDesktop }">
      <div class="scroll-progress" :style="{ width: scrollProgress + '%' }"></div>
      <button 
        class="back-button" 
        @click="$emit('back')"
        aria-label="Go back"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        >
          <path d="M19 12H5M12 19l-7-7 7-7" />
        </svg>
      </button>
      <h1 class="page-title">{{ title }}</h1>
      <button 
        class="bookmark-button" 
        @click="toggleBookmark"
        :class="{ 'bookmarked': isBookmarked }"
        :aria-label="isBookmarked ? 'Remove bookmark' : 'Add bookmark'"
      >
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="24"
          height="24"
          viewBox="0 0 24 24"
          :fill="isBookmarked ? 'currentColor' : 'none'"
          stroke="currentColor"
          stroke-width="2"
          class="bookmark-icon"
        >
          <path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z" />
        </svg>
        <span v-if="isDesktop" class="bookmark-label">
          Bookmark
        </span>
      </button>
    </div>

    <!-- Alert Notification -->
    <transition name="alert-fade">
      <div v-if="showBookmarkAlert" class="alert-notification" :class="alertType">
        <svg v-if="isBookmarked" xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
        <svg v-else xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M19 21l-7-5-7 5V5a2 2 0 0 1 2-2h10a2 2 0 0 1 2 2z"></path>
        </svg>
        <span>{{ bookmarkAlertMessage }}</span>
      </div>
    </transition>

    <!-- Success Toast Notification -->
    <transition name="alert-fade">
      <div v-if="showSuccessToast" class="alert-notification success-toast">
        <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
          <polyline points="20 6 9 17 4 12"></polyline>
        </svg>
        <span>{{ successToastMessage }}</span>
      </div>
    </transition>

    <!-- Content/Blocks -->
    <div class="content-area" ref="contentArea">
      <div v-for="(block, index) in blocks" :key="index" class="block">
        <!-- Hero Block -->
        <div v-if="block.type === 'hero'" class="hero-block">
          <div class="image-wrapper">
            <img 
              :src="block.imageUrl" 
              :alt="block.alt || `Hero image for ${title}`"
              @load="onImageLoad"
              @error="onImageError"
              :class="{ 'loaded': imageLoaded }"
              loading="lazy"
            />
            <div v-if="!imageLoaded" class="image-loading">
              <div class="loading-spinner"></div>
            </div>
          </div>
        </div>

        <!-- Text Block -->
        <div
          v-else-if="block.type === 'text'"
          class="text-block"
          v-html="block.content"
        ></div>

        <!-- Video Block -->
        <div v-else-if="block.type === 'video'" class="video-block">
          <div class="video-wrapper">
            <iframe
              :src="block.embedUrl"
              frameborder="0"
              allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
              allowfullscreen
              @load="onVideoLoad"
              :class="{ 'loaded': videoLoaded }"
            ></iframe>
            <div v-if="!videoLoaded" class="video-loading">
              <div class="loading-spinner"></div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Actions/CTA -->
    <div class="actions-section" :class="{ 'purchased': isPurchased }">
      <div v-for="(action, index) in actions" :key="index" class="action-item">
        <p v-if="action.pricing_text && !isPurchased" class="pricing-text">
          {{ action.pricing_text }}
        </p>
        <button 
          class="cta-button" 
          @click="handleAction(action)"
          :class="{ 
            'purchased': isPurchased,
            'pulse': !isPurchased
          }"
          :aria-label="isPurchased ? action.post_join_text : action.button_text"
          type="button"
        >
          {{ isPurchased ? action.post_join_text : action.button_text }}
          <svg 
            v-if="isPurchased" 
            xmlns="http://www.w3.org/2000/svg" 
            width="20" 
            height="20" 
            viewBox="0 0 24 24" 
            fill="none" 
            stroke="currentColor" 
            stroke-width="2.5"
          >
            <polyline points="20 6 9 17 4 12"></polyline>
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * Constants
 */
const TOAST_DURATION = 3000; // milliseconds
const DESKTOP_BREAKPOINT = 1200; // pixels
const SCROLL_RESET_DELAY = 0; // milliseconds

/**
 * GenericTemplate Component
 * 
 * A responsive template component that displays hero images, text blocks, and videos
 * with interactive features like bookmarking and CTA actions.
 * 
 * @component
 */
export default {
  name: "GenericTemplate",
  props: {
    /**
     * Page title displayed in the toolbar
     * @type {String}
     * @required
     */
    title: {
      type: String,
      required: true,
      validator: (value) => typeof value === 'string' && value.trim().length > 0,
    },
    /**
     * Array of content blocks (hero, text, video)
     * @type {Array}
     * @default []
     */
    blocks: {
      type: Array,
      default: () => [],
      validator: (value) => {
        if (!Array.isArray(value)) return false;
        return value.every(block => 
          block && typeof block === 'object' && 
          ['hero', 'text', 'video'].includes(block.type)
        );
      },
    },
    /**
     * Array of CTA actions with button text and pricing
     * @type {Array}
     * @default []
     */
    actions: {
      type: Array,
      default: () => [],
      validator: (value) => {
        if (!Array.isArray(value)) return false;
        return value.every(action => 
          action && typeof action === 'object' && 
          typeof action.button_text === 'string'
        );
      },
    },
  },
  data() {
    return {
      isBookmarked: false,
      isPurchased: false,
      isDesktop: false,
      imageLoaded: false,
      videoLoaded: false,
      resizeObserver: null,
      useResizeFallback: false,
      scrollProgress: 0,
      showBookmarkAlert: false,
      bookmarkAlertMessage: '',
      alertType: 'success',
      showSuccessToast: false,
      successToastMessage: '',
      successToastTimeout: null,
    };
  },
  computed: {
    /**
     * Generates a unique storage key based on the page title
     * @returns {String} Storage key for localStorage
     */
    storageKey() {
      return `page_${this.title.replace(/\s+/g, "_").toLowerCase()}`;
    },
  },
  mounted() {
    this.loadState();
    this.setupResizeObserver();
    this.checkDesktop();
    this.checkImageLoaded();
    this.setupScrollListener();
    // Listen for storage changes from other instances (mobile/desktop sync)
    window.addEventListener('storage', this.handleStorageChange);
    // Also listen for custom storage events (for same-window updates)
    window.addEventListener('localStorageUpdate', this.handleLocalStorageUpdate);
  },
  beforeDestroy() {
    if (this.resizeObserver) {
      this.resizeObserver.disconnect();
    }
    if (this.useResizeFallback) {
      window.removeEventListener('resize', this.checkDesktop);
    }
    this.removeScrollListener();
    window.removeEventListener('storage', this.handleStorageChange);
    window.removeEventListener('localStorageUpdate', this.handleLocalStorageUpdate);
    // Clear success toast timeout
    if (this.successToastTimeout) {
      clearTimeout(this.successToastTimeout);
    }
  },
  watch: {
    title() {
      // When switching pages, reload the state
      this.loadState();
      this.imageLoaded = false;
      this.videoLoaded = false;
      this.scrollProgress = 0; // Reset scroll progress when switching tabs
      this.$nextTick(() => {
        this.checkImageLoaded();
        // Reset scroll position
        if (this.$refs.contentArea) {
          this.$refs.contentArea.scrollTop = 0;
        }
      });
    },
    blocks: {
      handler() {
        // Reset image loaded state when blocks change
        this.imageLoaded = false;
        this.videoLoaded = false;
        this.scrollProgress = 0; // Reset scroll progress when blocks change
        this.$nextTick(() => {
          this.checkImageLoaded();
          // Reset scroll position
          if (this.$refs.contentArea) {
            this.$refs.contentArea.scrollTop = 0;
          }
        });
      },
      deep: true,
    },
  },
  methods: {
    // ==================== Lifecycle Methods ====================
    
    /**
     * Sets up ResizeObserver to detect desktop/mobile breakpoint
     * Falls back to window resize listener if ResizeObserver is not available
     */
    setupResizeObserver() {
      if (typeof ResizeObserver !== 'undefined') {
        this.resizeObserver = new ResizeObserver(() => {
          this.checkDesktop();
        });
        this.$nextTick(() => {
          if (this.$el) {
            this.resizeObserver.observe(this.$el);
          }
        });
      } else {
        // Fallback for browsers without ResizeObserver
        this.useResizeFallback = true;
        window.addEventListener('resize', this.checkDesktop);
        this.checkDesktop();
      }
    },
    
    /**
     * Checks if the current viewport is desktop size
     * Uses component width or falls back to window width
     */
    checkDesktop() {
      if (this.$el) {
        const width = this.$el.offsetWidth || this.$el.clientWidth;
        this.isDesktop = width >= DESKTOP_BREAKPOINT;
      } else {
        // Fallback: check window width
        this.isDesktop = window.innerWidth >= DESKTOP_BREAKPOINT;
      }
    },
    // ==================== State Management ====================
    
    /**
     * Loads saved state from localStorage
     * Handles errors gracefully if localStorage is unavailable or corrupted
     */
    loadState() {
      try {
        const saved = localStorage.getItem(this.storageKey);
        if (saved) {
          const state = JSON.parse(saved);
          this.isBookmarked = Boolean(state.isBookmarked);
          this.isPurchased = Boolean(state.isPurchased);
        } else {
          this.isBookmarked = false;
          this.isPurchased = false;
        }
      } catch (error) {
        console.error('Error loading state from localStorage:', error);
        // Reset to default state on error
        this.isBookmarked = false;
        this.isPurchased = false;
      }
      
      // Don't show success toast when loading saved state
      this.showSuccessToast = false;
      if (this.successToastTimeout) {
        clearTimeout(this.successToastTimeout);
        this.successToastTimeout = null;
      }
    },
    
    /**
     * Saves current state to localStorage and syncs with other instances
     * @throws {Error} If localStorage is unavailable
     */
    saveState() {
      try {
        const state = {
          isBookmarked: this.isBookmarked,
          isPurchased: this.isPurchased,
        };
        localStorage.setItem(this.storageKey, JSON.stringify(state));
        
        // Dispatch custom event for same-window sync
        // (storage event only fires in other windows/tabs)
        window.dispatchEvent(new CustomEvent('localStorageUpdate', {
          detail: { key: this.storageKey, state }
        }));
      } catch (error) {
        console.error('Error saving state to localStorage:', error);
      }
    },
    /**
     * Handles storage events from other windows/tabs
     * @param {StorageEvent} event - Browser storage event
     */
    handleStorageChange(event) {
      if (event.key === this.storageKey && event.newValue) {
        try {
          const state = JSON.parse(event.newValue);
          this.isBookmarked = Boolean(state.isBookmarked);
          this.isPurchased = Boolean(state.isPurchased);
        } catch (error) {
          console.error('Error parsing storage state:', error);
        }
      }
    },
    
    /**
     * Handles custom localStorage update events from same window
     * Used for syncing between mobile and desktop previews
     * @param {CustomEvent} event - Custom localStorage update event
     */
    handleLocalStorageUpdate(event) {
      if (event.detail && event.detail.key === this.storageKey) {
        const state = event.detail.state;
        this.isBookmarked = Boolean(state.isBookmarked);
        this.isPurchased = Boolean(state.isPurchased);
        // Don't show alert on sync - only show when user clicks
      }
    },
    // ==================== User Actions ====================
    
    /**
     * Toggles bookmark state and shows alert notification
     */
    toggleBookmark() {
      this.isBookmarked = !this.isBookmarked;
      this.saveState();
      this.showBookmarkAlertMessage();
    },
    
    /**
     * Shows bookmark alert notification with appropriate message
     */
    showBookmarkAlertMessage() {
      this.bookmarkAlertMessage = this.isBookmarked 
        ? 'Bookmark saved' 
        : 'Bookmark removed';
      this.alertType = 'success';
      this.showBookmarkAlert = true;
      
      // Hide alert after configured duration
      setTimeout(() => {
        this.showBookmarkAlert = false;
      }, TOAST_DURATION);
    },
    
    /**
     * Handles CTA button click action
     * Toggles purchased state and shows success toast
     * @param {Object} action - Action object with button text and pricing
     */
    handleAction(action) {
      if (!action || typeof action !== 'object') {
        console.warn('Invalid action provided to handleAction');
        return;
      }
      
      this.isPurchased = !this.isPurchased;
      this.saveState();
      
      // Show success toast when purchased
      if (this.isPurchased) {
        this.showSuccessToastMessage(action);
      } else {
        // Clear toast when toggled off
        this.showSuccessToast = false;
        if (this.successToastTimeout) {
          clearTimeout(this.successToastTimeout);
          this.successToastTimeout = null;
        }
      }
    },
    
    /**
     * Shows success toast notification with action message
     * @param {Object} action - Action object containing post_join_text
     */
    showSuccessToastMessage(action) {
      this.successToastMessage = action.post_join_text || 'Success!';
      this.showSuccessToast = true;
      
      // Clear any existing timeout
      if (this.successToastTimeout) {
        clearTimeout(this.successToastTimeout);
      }
      
      // Hide toast after configured duration
      this.successToastTimeout = setTimeout(() => {
        this.showSuccessToast = false;
      }, TOAST_DURATION);
    },
    // ==================== Media Loading ====================
    
    /**
     * Checks if hero images are already loaded (e.g., from cache)
     */
    checkImageLoaded() {
      this.$nextTick(() => {
        const heroBlocks = this.blocks.filter(block => block.type === 'hero');
        if (heroBlocks.length > 0 && this.$el) {
          const images = this.$el.querySelectorAll('.hero-block img');
          if (images.length > 0) {
            const firstImage = images[0];
            if (firstImage && firstImage.complete && firstImage.naturalHeight > 0) {
              this.imageLoaded = true;
            }
          }
        }
      });
    },
    
    /**
     * Handles image load event
     */
    onImageLoad() {
      this.imageLoaded = true;
    },
    
    /**
     * Handles image error event (treats as loaded to hide spinner)
     */
    onImageError() {
      this.imageLoaded = true;
    },
    
    /**
     * Handles video iframe load event
     */
    onVideoLoad() {
      this.videoLoaded = true;
    },
    // ==================== Scroll Management ====================
    
    /**
     * Sets up scroll listener for progress indicator
     */
    setupScrollListener() {
      if (this.$refs.contentArea) {
        this.$refs.contentArea.addEventListener('scroll', this.handleScroll);
        this.handleScroll(); // Initial calculation
      }
    },
    
    /**
     * Removes scroll listener to prevent memory leaks
     */
    removeScrollListener() {
      if (this.$refs.contentArea) {
        this.$refs.contentArea.removeEventListener('scroll', this.handleScroll);
      }
    },
    
    /**
     * Calculates and updates scroll progress percentage
     */
    handleScroll() {
      if (this.$refs.contentArea) {
        const element = this.$refs.contentArea;
        const scrollTop = element.scrollTop;
        const scrollHeight = element.scrollHeight - element.clientHeight;
        this.scrollProgress = scrollHeight > 0 
          ? (scrollTop / scrollHeight) * 100 
          : 0;
      }
    },
  },
};
</script>

<style scoped>
.generic-template {
  display: flex;
  flex-direction: column;
  height: 100%;
  background-color: var(--color-bg);
  position: relative;
  overflow: visible;
  z-index: 1;
}

/* Toolbar - Mobile First */
.toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 64px;
  padding: 0 20px;
  background-color: var(--color-bg);
  border-bottom: 1px solid var(--color-border);
  position: sticky;
  top: 0;
  z-index: 10;
  backdrop-filter: blur(10px);
  background-color: rgba(255, 255, 255, 0.95);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05);
  overflow: visible;
}

/* Scroll Progress Indicator */
.scroll-progress {
  position: absolute;
  bottom: 0;
  left: 0;
  height: 3px;
  background: linear-gradient(90deg, var(--color-primary) 0%, #1d4ed8 100%);
  transition: width 0.1s ease-out;
  z-index: 11;
  box-shadow: 0 0 8px rgba(37, 99, 235, 0.4);
  width: 100%;
}

/* Desktop Toolbar */
.toolbar-desktop {
  height: 72px;
  padding: 0;
  width: 100%;
  border-bottom: 1px solid rgba(229, 231, 235, 0.8);
  position: relative;
}

.toolbar-desktop .back-button,
.toolbar-desktop .bookmark-button {
  position: absolute;
  z-index: 2;
  padding: 10px;
}

.toolbar-desktop .back-button {
  left: 40px;
  opacity: 0.6;
  transition: opacity 0.2s ease;
}

.toolbar-desktop .back-button:hover {
  opacity: 1;
}

.toolbar-desktop .bookmark-button {
  right: 40px;
}

.toolbar-desktop .page-title {
  max-width: 1200px;
  width: 100%;
  margin: 0 auto;
  padding: 0 120px;
  text-align: center;
}

.back-button,
.bookmark-button {
  background: none;
  border: none;
  cursor: pointer;
  padding: 10px;
  color: var(--color-text);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  border-radius: 8px;
  display: flex;
  align-items: center;
  gap: 8px;
  position: relative;
  z-index: 10;
  overflow: visible;
}

.back-button:hover,
.bookmark-button:hover {
  background-color: rgba(0, 0, 0, 0.06);
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.back-button:active,
.bookmark-button:active {
  transform: translateY(0) scale(0.96);
  background-color: rgba(0, 0, 0, 0.08);
  transition: transform 0.1s ease, background-color 0.1s ease;
}

.back-button:focus-visible,
.bookmark-button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
  border-radius: 8px;
}

.bookmark-button {
  font-weight: 500;
  font-size: 14px;
}

.bookmark-button.bookmarked {
  color: var(--color-primary);
}

.bookmark-icon {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.bookmark-button.bookmarked .bookmark-icon {
  transform: scale(1.1);
  animation: bookmarkPop 0.4s cubic-bezier(0.4, 0, 0.2, 1);
}

@keyframes bookmarkPop {
  0%, 100% { transform: scale(1.1); }
  50% { transform: scale(1.3) rotate(5deg); }
}

.bookmark-label {
  font-size: 14px;
  font-weight: 500;
}

/* Alert Notification */
.alert-notification {
  position: absolute;
  top: 80px;
  left: 50%;
  transform: translateX(-50%);
  background-color: white;
  color: #1f2937;
  padding: 12px 20px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 14px;
  font-weight: 500;
  z-index: 10000;
  min-width: 200px;
  max-width: 90%;
  text-align: center;
  justify-content: center;
  pointer-events: none;
}

.alert-notification.success,
.alert-notification.success-toast {
  background-color: #10b981;
  color: white;
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);
}

.alert-notification svg {
  flex-shrink: 0;
}

/* Alert fade transitions */
.alert-fade-enter-active,
.alert-fade-leave-active {
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.alert-fade-enter-from {
  opacity: 0;
  transform: translateX(-50%) translateY(-20px);
}

.alert-fade-leave-to {
  opacity: 0;
  transform: translateX(-50%) translateY(-20px);
}

/* Desktop positioning */
.is-desktop .alert-notification {
  top: 90px;
  max-width: 400px;
}


.page-title {
  font-size: 18px;
  font-weight: 600;
  color: var(--color-text);
  flex: 1;
  text-align: center;
  transition: all 0.3s ease;
}

.toolbar-desktop .page-title {
  font-size: 20px;
  font-weight: 700;
  letter-spacing: -0.02em;
}

/* Content Area */
.content-area {
  flex: 1;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  scroll-behavior: smooth;
  scroll-padding-top: 20px;
}

.content-area::-webkit-scrollbar {
  width: 6px;
}

.content-area::-webkit-scrollbar-track {
  background: transparent;
  transition: background 0.3s ease;
}

.content-area::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 3px;
  transition: background 0.3s ease;
}

.content-area::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 0, 0, 0.35);
}

.content-area:focus-within {
  outline: none;
}

.block {
  margin-bottom: 24px;
  animation: fadeInUp 0.6s cubic-bezier(0.4, 0, 0.2, 1);
  animation-fill-mode: both;
}

.block:nth-child(1) { animation-delay: 0s; }
.block:nth-child(2) { animation-delay: 0.1s; }
.block:nth-child(3) { animation-delay: 0.2s; }
.block:nth-child(4) { animation-delay: 0.3s; }
.block:nth-child(5) { animation-delay: 0.4s; }

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* Hero Block */
.hero-block {
  margin-bottom: 0;
}

.image-wrapper {
  position: relative;
  width: 100%;
  overflow: hidden;
  background: linear-gradient(135deg, #f5f7fa 0%, #e9ecef 100%);
}

.image-wrapper img {
  width: 100%;
  height: auto;
  display: block;
  opacity: 0;
  transition: opacity 0.6s cubic-bezier(0.4, 0, 0.2, 1), transform 0.6s cubic-bezier(0.4, 0, 0.2, 1);
  object-fit: cover;
  transform: scale(1.02);
  position: relative;
  z-index: 1;
}

.image-wrapper img.loaded {
  opacity: 1;
  transform: scale(1);
  z-index: 2;
}

.image-wrapper:hover img.loaded {
  transform: scale(1.01);
  transition: transform 0.3s ease;
}

.image-loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
}

.video-loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1;
}

.loading-spinner {
  width: 48px;
  height: 48px;
  border: 4px solid rgba(37, 99, 235, 0.15);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
  position: relative;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(4px);
}

.loading-spinner::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 20px;
  height: 20px;
  margin: -10px 0 0 -10px;
  border: 3px solid rgba(37, 99, 235, 0.25);
  border-top-color: var(--color-primary);
  border-radius: 50%;
  animation: spin 0.6s linear infinite reverse;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* Enhanced loading states */
.image-loading,
.video-loading {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

/* Text Block */
.text-block {
  padding: 0 20px;
  line-height: 1.7;
  max-width: 100%;
}

.is-desktop .text-block {
  max-width: 80%;
  margin: 0 auto;
  padding: 0 40px;
}

.text-block ::v-deep h2 {
  font-size: 28px;
  font-weight: 700;
  margin-bottom: 20px;
  margin-top: 12px;
  color: var(--color-text);
  line-height: 1.25;
  letter-spacing: -0.03em;
  transition: color 0.3s ease;
}

.is-desktop .text-block ::v-deep h2 {
  font-size: 38px;
  margin-bottom: 24px;
  line-height: 1.2;
}

.text-block ::v-deep h3 {
  font-size: 22px;
  font-weight: 600;
  margin-bottom: 14px;
  margin-top: 28px;
  color: var(--color-text);
  letter-spacing: -0.02em;
  line-height: 1.3;
}

.is-desktop .text-block ::v-deep h3 {
  font-size: 28px;
  margin-bottom: 18px;
  margin-top: 32px;
}

.text-block ::v-deep p {
  margin-bottom: 18px;
  color: #4b5563;
  font-size: 16px;
  line-height: 1.7;
  transition: color 0.3s ease;
}

.is-desktop .text-block ::v-deep p {
  font-size: 18px;
  line-height: 1.8;
  margin-bottom: 20px;
}

.text-block ::v-deep ul {
  margin-left: 24px;
  margin-bottom: 18px;
  list-style-type: disc;
  padding-left: 4px;
}

.text-block ::v-deep li {
  margin-bottom: 12px;
  color: #4b5563;
  font-size: 16px;
  line-height: 1.65;
  transition: color 0.3s ease;
}

.is-desktop .text-block ::v-deep li {
  font-size: 18px;
  line-height: 1.7;
  margin-bottom: 14px;
}

.text-block ::v-deep h1 {
  font-size: 32px;
  font-weight: 700;
  margin-bottom: 20px;
  margin-top: 12px;
  color: var(--color-text);
  line-height: 1.2;
  letter-spacing: -0.04em;
}

.is-desktop .text-block ::v-deep h1 {
  font-size: 44px;
  margin-bottom: 24px;
}

.text-block ::v-deep h4,
.text-block ::v-deep h5,
.text-block ::v-deep h6 {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 12px;
  margin-top: 20px;
  color: var(--color-text);
  line-height: 1.4;
}

.is-desktop .text-block ::v-deep h4,
.is-desktop .text-block ::v-deep h5,
.is-desktop .text-block ::v-deep h6 {
  font-size: 22px;
  margin-bottom: 16px;
  margin-top: 24px;
}

.text-block ::v-deep strong {
  font-weight: 600;
  color: var(--color-text);
}

.text-block ::v-deep em {
  font-style: italic;
}

.text-block ::v-deep a {
  color: #3b82f6;
  text-decoration: underline;
  transition: color 0.2s ease;
}

.text-block ::v-deep a:hover {
  color: #2563eb;
}

/* Video Block */
.video-block {
  padding: 0 20px;
  margin-bottom: 24px;
}

.is-desktop .video-block {
  max-width: 680px;
  margin: 0 auto 24px;
  padding: 0 40px;
}

.video-wrapper {
  position: relative;
  width: 100%;
  padding-bottom: 56.25%; /* 16:9 aspect ratio */
  height: 0;
  overflow: hidden;
  border-radius: 12px;
  background: linear-gradient(135deg, #f5f7fa 0%, #e9ecef 100%);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.video-wrapper:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
}

.video-wrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: none;
  border-radius: 12px;
  opacity: 0;
  transition: opacity 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.video-wrapper iframe.loaded {
  opacity: 1;
}

/* Actions Section - Enhanced CTA */
.actions-section {
  padding: 24px 20px calc(24px + env(safe-area-inset-bottom));
  background: linear-gradient(to top, rgba(255, 255, 255, 1) 0%, rgba(255, 255, 255, 0.98) 100%);
  border-top: 1px solid rgba(229, 231, 235, 0.8);
  display: flex;
  flex-direction: column;
  justify-content: center;
  position: sticky;
  bottom: 0;
  z-index: 10;
  backdrop-filter: blur(20px);
  box-shadow: 0 -2px 16px rgba(0, 0, 0, 0.04);
  transition: all 0.3s ease;
  overflow: visible;
}

.is-desktop .actions-section {
  width: 100%;
  padding: 32px 40px;
  border-radius: 0;
  border-top: 1px solid rgba(229, 231, 235, 0.6);
}

.actions-section.purchased {
  background: linear-gradient(to top, rgba(16, 185, 129, 0.08) 0%, rgba(255, 255, 255, 0.98) 100%);
  border-top-color: rgba(16, 185, 129, 0.3);
  animation: purchasedGlow 0.6s ease-out;
}

@keyframes purchasedGlow {
  0% {
    box-shadow: 0 -4px 20px rgba(0, 0, 0, 0.05);
  }
  50% {
    box-shadow: 0 -4px 30px rgba(16, 185, 129, 0.2);
  }
  100% {
    box-shadow: 0 -4px 20px rgba(0, 0, 0, 0.05);
  }
}

.action-item {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.is-desktop .action-item {
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap: 24px;
  max-width: 1200px;
  margin: 0 auto;
}

.pricing-text {
  font-size: 15px;
  color: #6b7280;
  font-weight: 500;
  margin: 0;
  text-align: center;
}

.is-desktop .pricing-text {
  font-size: 17px;
  color: #374151;
  font-weight: 600;
}


@keyframes slideInDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes checkmarkDraw {
  0% {
    stroke-dasharray: 0 50;
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    stroke-dasharray: 50 0;
  }
}

.cta-button {
  width: 100%;
  padding: 16px 24px;
  background: linear-gradient(135deg, var(--color-primary) 0%, #1d4ed8 100%);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  box-shadow: 0 4px 14px rgba(37, 99, 235, 0.25), 0 2px 4px rgba(37, 99, 235, 0.1);
  position: relative;
  overflow: visible;
  letter-spacing: 0.01em;
}

.is-desktop .cta-button {
  padding: 18px 48px;
  font-size: 17px;
  border-radius: 14px;
  width: auto;
  min-width: 240px;
  flex-shrink: 0;
}

.cta-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s;
}

.cta-button:hover::before {
  left: 100%;
}

.cta-button:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(37, 99, 235, 0.45);
  background: linear-gradient(135deg, #1d4ed8 0%, var(--color-primary) 100%);
}

.cta-button:active {
  transform: translateY(-1px) scale(0.98);
  box-shadow: 0 4px 16px rgba(37, 99, 235, 0.35);
  transition: transform 0.1s ease, box-shadow 0.1s ease;
}

.cta-button:focus-visible {
  outline: 3px solid rgba(37, 99, 235, 0.4);
  outline-offset: 3px;
  border-radius: 12px;
}

.cta-button.pulse {
  animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes pulse {
  0%, 100% {
    box-shadow: 0 4px 12px rgba(37, 99, 235, 0.3);
  }
  50% {
    box-shadow: 0 4px 20px rgba(37, 99, 235, 0.5);
  }
}

.cta-button.purchased {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
  box-shadow: 0 4px 12px rgba(16, 185, 129, 0.3);
  animation: none;
  overflow: hidden;
}

.cta-button.purchased:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(16, 185, 129, 0.45);
  background: linear-gradient(135deg, #059669 0%, #10b981 100%);
}

.cta-button.purchased:active {
  transform: translateY(-1px) scale(0.98);
  box-shadow: 0 4px 16px rgba(16, 185, 129, 0.35);
}


@keyframes checkmarkPop {
  0% {
    opacity: 0;
    transform: scale(0) rotate(-45deg);
  }
  50% {
    transform: scale(1.2) rotate(5deg);
  }
  100% {
    opacity: 1;
    transform: scale(1) rotate(0deg);
  }
}

/* Responsive adjustments */
.is-desktop .hero-block {
  margin-bottom: 32px;
}

.is-desktop .content-area .block {
  margin-bottom: 32px;
}
</style>
