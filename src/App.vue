<template>
  <div id="app">
    <!-- Tab Navigation -->
    <div class="tab-navigation">
      <button
        v-for="tab in tabs"
        :key="tab.id"
        :class="['tab-button', { active: activeTab === tab.id }]"
        @click="handleTabChange(tab.id)"
        :aria-label="`Switch to ${tab.label} page`"
        :aria-pressed="activeTab === tab.id"
        type="button"
      >
        {{ tab.label }}
      </button>
    </div>

    <!-- Preview Panels -->
    <div class="preview-container">
      <!-- Mobile Preview -->
      <div class="preview-panel mobile-preview">
        <div class="preview-label">Mobile Preview (375px)</div>
        <div class="mobile-frame">
          <GenericTemplate
            :title="currentPageData.title"
            :blocks="currentPageData.blocks"
            :actions="currentPageData.actions"
          />
        </div>
      </div>

      <!-- Desktop Preview -->
      <div class="preview-panel desktop-preview">
        <div class="preview-label">Desktop Preview</div>
        <div class="desktop-frame">
          <GenericTemplate
            :title="currentPageData.title"
            :blocks="currentPageData.blocks"
            :actions="currentPageData.actions"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import GenericTemplate from './components/GenericTemplate.vue'

/**
 * Main App Component
 * 
 * Displays side-by-side mobile and desktop previews of the GenericTemplate component
 * with tab navigation for different page types.
 * 
 * @component
 */
export default {
  name: 'App',
  components: {
    GenericTemplate,
  },
  data() {
    return {
      activeTab: 'program',
      tabs: [
        { id: 'program', label: 'Program' },
        { id: 'event', label: 'Event' },
        { id: 'sales', label: 'Sales Page' }
      ],
      pageData: {
        program: {
          title: "Amari Fitness Program",
          actions: [
            {
              button_text: "Join Program",
              pricing_text: "From $197/month",
              post_join_text: "Joined"
            }
          ],
          blocks: [
            { type: 'hero', imageUrl: 'https://images.unsplash.com/photo-1571019614242-c5c5dee9f50b?w=800' },
            {
              type: 'text',
              content: '<h2>4 weeks. Every body. Every goal.</h2><p>Transform your fitness journey with our comprehensive program designed for all fitness levels. Get access to workouts, nutrition guides, and community support.</p>'
            },
            {
              type: 'text',
              content: '<h3>What You Get</h3><ul><li>4 weeks of structured workouts</li><li>Nutrition guidance</li><li>Community access</li><li>Progress tracking</li><li>Expert support</li></ul>'
            },
            {
              type: 'text',
              content: '<h3>About the Program</h3><p>Our program is built on proven principles of progressive overload and sustainable habit formation. Whether you\'re just starting out or looking to level up, we meet you where you are.</p>'
            }
          ]
        },
        event: {
          title: "Live Yoga Session",
          actions: [
            {
              button_text: "Register Now",
              pricing_text: "Free",
              post_join_text: "Registered"
            }
          ],
          blocks: [
            { type: 'hero', imageUrl: 'https://images.unsplash.com/photo-1544367567-0f2fcb009e0b?w=800' },
            {
              type: 'text',
              content: '<h2>Join us for a live session</h2><p>Saturday, November 9th at 10:00 AM PST. A 60-minute flow suitable for all levels.</p>'
            },
            { type: 'video', embedUrl: 'https://www.youtube.com/embed/v7AYKMP6rOE' },
            {
              type: 'text',
              content: '<h3>What to Bring</h3><ul><li>Yoga mat</li><li>Water bottle</li><li>Comfortable clothes</li><li>Optional: blocks and strap</li></ul>'
            }
          ]
        },
        sales: {
          title: "Premium Membership",
          actions: [
            {
              button_text: "Start Free Trial",
              pricing_text: "Then $29/month",
              post_join_text: "Trial Started"
            }
          ],
          blocks: [
            { type: 'hero', imageUrl: 'https://images.unsplash.com/photo-1534438327276-14e5300c3a48?w=800' },
            {
              type: 'text',
              content: '<h2>Everything you need to succeed</h2><p>Get unlimited access to all programs, live classes, and premium content.</p>'
            },
            {
              type: 'text',
              content: '<h3>Premium Features</h3><ul><li>Access to all programs</li><li>Unlimited live classes</li><li>1-on-1 coaching sessions</li><li>Custom meal plans</li><li>Priority support</li></ul>'
            },
            {
              type: 'text',
              content: '<h3>Ready to start?</h3><p>Try it free for 14 days. Cancel anytime, no questions asked.</p>'
            }
          ]
        }
      }
    }
  },
  computed: {
    /**
     * Gets the current page data based on active tab
     * @returns {Object} Page data object with title, blocks, and actions
     */
    currentPageData() {
      return this.pageData[this.activeTab] || this.pageData.program
    },
  },
  methods: {
    /**
     * Handles tab change event
     * @param {String} tabId - ID of the tab to switch to
     */
    handleTabChange(tabId) {
      if (this.pageData[tabId]) {
        this.activeTab = tabId
      }
    },
  },
}
</script>

<style scoped>
#app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

/* Tab Navigation */
.tab-navigation {
  display: flex;
  gap: 8px;
  padding: 20px;
  background-color: white;
  border-bottom: 1px solid var(--color-border);
}

.tab-button {
  padding: 10px 24px;
  background-color: white;
  border: 1px solid var(--color-border);
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
  color: var(--color-text);
  outline: none;
}

.tab-button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.tab-button:hover {
  background-color: #f9fafb;
}

.tab-button.active {
  background-color: var(--color-primary);
  color: white;
  border-color: var(--color-primary);
}

/* Preview Container */
.preview-container {
  display: flex;
  flex: 1;
  overflow: hidden;
  background-color: #f9fafb;
}

.preview-panel {
  display: flex;
  flex-direction: column;
  padding: 24px;
}

.preview-label {
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
  color: #6b7280;
  margin-bottom: 12px;
  letter-spacing: 0.05em;
}

/* Mobile Preview */
.mobile-preview {
  width: 450px;
  min-width: 450px;
  border-right: 1px solid var(--color-border);
  overflow: hidden;
  position: relative;
}

.mobile-frame {
  width: 375px;
  height: 667px;
  background-color: white;
  border-radius: 20px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  z-index: 0;
  position: relative;
}

/* Desktop Preview */
.desktop-preview {
  flex: 1;
  min-width: 0;
}

.desktop-frame {
  width: 100%;
  height: 100%;
  min-width: 1200px;
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  overflow: hidden;
}
</style>
