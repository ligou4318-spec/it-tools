<script setup lang="ts">
import { IconSearch, IconCommand, IconStar, IconRocket, IconBrandGithub, IconWorld } from '@tabler/icons-vue';
import { useI18n } from 'vue-i18n';
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';
import { useToolStore } from '@/tools/tools.store';

const { t } = useI18n();
const router = useRouter();
const toolStore = useToolStore();

const searchQuery = ref('');
const isSearchFocused = ref(false);
const showCommandPalette = ref(false);

// Keyboard shortcut ⌘K / Ctrl+K
function handleKeydown(e: KeyboardEvent) {
  if ((e.metaKey || e.ctrlKey) && e.key === 'k') {
    e.preventDefault();
    showCommandPalette.value = !showCommandPalette.value;
    if (showCommandPalette.value) {
      setTimeout(() => {
        const input = document.querySelector('.sv-search-input') as HTMLInputElement;
        input?.focus();
      }, 100);
    }
  }
  if (e.key === 'Escape' && showCommandPalette.value) {
    showCommandPalette.value = false;
  }
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
});

// Get all tools for search
const filteredTools = computed(() => {
  const tools = toolStore.tools || [];
  if (!searchQuery.value) {
    return tools.slice(0, 6); // Show top 6 tools initially
  }
  const query = searchQuery.value.toLowerCase();
  return tools.filter(tool => {
    const nameMatch = tool.name?.toLowerCase().includes(query);
    const descMatch = tool.description?.toLowerCase().includes(query);
    const pathMatch = tool.path?.toLowerCase().includes(query);
    return nameMatch || descMatch || pathMatch;
  }).slice(0, 8);
});

function navigateToTool(tool: any) {
  if (tool.path) {
    router.push(`/${tool.path}`);
  }
  searchQuery.value = '';
  showCommandPalette.value = false;
}

function handleExplore() {
  window.scrollTo({ top: 500, behavior: 'smooth' });
}

function goToGitHub() {
  window.open('https://github.com/CorentinTh/it-tools', '_blank');
}
</script>

<template>
  <section class="sv-hero">
    <div class="sv-container">
      <!-- Badge -->
      <div class="sv-badge">
        <IconRocket :size="14" />
        <span>v2026.1 • Powered by AI • Open Source</span>
      </div>

      <!-- Main Heading -->
      <h1 class="sv-title">
        The Developer's <span>Utility Belt</span>
      </h1>

      <!-- Subtitle -->
      <p class="sv-desc">
        All-in-one suite for modern engineers. Local-first, zero tracking, and built for
        <span class="highlight">developers worldwide</span>. From JSON formatting to cron jobs,
        get it all here.
      </p>

      <!-- Search Box - GitHub Style -->
      <div class="sv-search-box" :class="{ 'sv-search-focused': isSearchFocused || showCommandPalette }">
        <div class="kdb-shortcut">
          <IconCommand :size="12" />
          <span>K</span>
        </div>
        <input
          v-model="searchQuery"
          type="text"
          class="sv-search-input"
          placeholder="I want to... (e.g. format JSON, debug JWT, generate cron)"
          @focus="isSearchFocused = true; showCommandPalette = true"
          @blur="setTimeout(() => isSearchFocused = false, 200)"
          @keydown.enter="filteredTools[0] && navigateToTool(filteredTools[0])"
          @keydown.escape="showCommandPalette = false"
        />
        <button class="sv-btn-go" @click="handleExplore">
          <IconSearch :size="18" />
        </button>

        <!-- Command Palette Results -->
        <div v-if="showCommandPalette" class="sv-command-palette">
          <div class="sv-command-header">
            <span>Quick Actions</span>
            <span class="sv-command-hint"><IconCommand :size="12"/> K to close</span>
          </div>
          <div class="sv-command-results">
            <div
              v-for="tool in filteredTools"
              :key="tool.name"
              class="sv-command-item"
              @mousedown="navigateToTool(tool)"
            >
              <div class="sv-command-icon">
                <IconRocket :size="16" />
              </div>
              <div class="sv-command-content">
                <div class="sv-command-title">{{ tool.name }}</div>
                <div class="sv-command-desc">{{ tool.description || 'Quick access tool' }}</div>
              </div>
            </div>
            <div v-if="filteredTools.length === 0" class="sv-command-empty">
              No tools found. Try a different search term.
            </div>
          </div>
        </div>
      </div>

      <!-- Ecosystem -->
      <div class="sv-ecosystem">
        <p>Seamlessly integrates with your workflow:</p>
        <div class="sv-icons">
          <span class="sv-ecosystem-badge"><IconBrandGithub :size="14"/> GitHub</span>
          <span class="sv-ecosystem-badge"><IconWorld :size="14"/> Web</span>
          <span class="sv-ecosystem-badge">VS Code</span>
          <span class="sv-ecosystem-badge">Docker</span>
        </div>
      </div>

      <!-- Social Proof -->
      <div class="sv-social-proof">
        <div class="sv-stars">
          <IconStar :size="16" />
          <span>36.3k GitHub Stars</span>
        </div>
        <div class="sv-divider">•</div>
        <div class="sv-featured">
          Featured on:
          <span>Product Hunt</span>
          <span>Hacker News</span>
        </div>
      </div>
    </div>
  </section>
</template>

<style lang="less" scoped>
// GitHub-inspired dark theme variables
:root {
  --sv-bg: #0B0E14;
  --sv-accent: #38BDF8;
  --sv-text: #E2E8F0;
  --sv-card: #161B22;
  --sv-border: #30363D;
  --sv-muted: #8B949E;
  --sv-bg-secondary: #21262D;
}

.sv-hero {
  background-color: var(--sv-bg);
  color: var(--sv-text);
  padding: 120px 20px;
  display: flex;
  justify-content: center;
  font-family: 'JetBrains Mono', 'Inter', monospace;
  position: relative;
  overflow: hidden;

  // Subtle background pattern
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-image:
      linear-gradient(rgba(56, 189, 248, 0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(56, 189, 248, 0.03) 1px, transparent 1px);
    background-size: 50px 50px;
    opacity: 0.5;
    pointer-events: none;
  }
}

.sv-container {
  max-width: 1200px;
  width: 100%;
  text-align: center;
  position: relative;
  z-index: 1;
}

.sv-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px 16px;
  background: var(--sv-bg-secondary);
  border: 1px solid var(--sv-border);
  border-radius: 24px;
  color: var(--sv-accent);
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 32px;
  animation: fadeInDown 0.6s ease-out;
}

.sv-title {
  font-size: 4rem;
  letter-spacing: -0.02em;
  font-weight: 800;
  margin-bottom: 20px;
  line-height: 1.1;
  animation: fadeInUp 0.6s ease-out 0.2s both;
  color: var(--sv-text);

  span {
    color: var(--sv-accent);
    text-decoration: underline;
    text-decoration-style: wavy;
    text-decoration-color: rgba(56, 189, 248, 0.3);
    text-underline-offset: 8px;
  }
}

.sv-desc {
  font-size: 1.125rem;
  color: var(--sv-muted);
  line-height: 1.7;
  margin-bottom: 45px;
  max-width: 700px;
  margin-left: auto;
  margin-right: auto;
  font-weight: 400;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  animation: fadeInUp 0.6s ease-out 0.4s both;

  .highlight {
    color: var(--sv-accent);
    font-weight: 500;
  }
}

// GitHub-style search box
.sv-search-box {
  position: relative;
  background: var(--sv-card);
  border: 1px solid var(--sv-border);
  border-radius: 12px;
  display: flex;
  align-items: center;
  padding: 12px 20px;
  max-width: 700px;
  margin: 40px auto;
  transition: border-color 0.2s, box-shadow 0.2s;
  animation: fadeInUp 0.6s ease-out 0.6s both;

  &:focus-within {
    border-color: var(--sv-accent);
    box-shadow: 0 0 0 3px rgba(56, 189, 248, 0.1);
  }

  &.sv-search-focused {
    border-color: var(--sv-accent);
  }
}

.kdb-shortcut {
  background: var(--sv-bg-secondary);
  border: 1px solid var(--sv-border);
  padding: 4px 8px;
  border-radius: 6px;
  font-size: 12px;
  color: var(--sv-muted);
  margin-right: 12px;
  display: flex;
  align-items: center;
  gap: 4px;
  font-family: 'JetBrains Mono', monospace;
  font-weight: 500;
  flex-shrink: 0;
}

.sv-search-input {
  flex: 1;
  background: transparent;
  border: none;
  color: var(--sv-text);
  font-size: 15px;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  outline: none;
  padding: 8px 0;

  &::placeholder {
    color: var(--sv-muted);
  }
}

.sv-btn-go {
  background: var(--sv-accent);
  color: var(--sv-bg);
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.15s ease;
  flex-shrink: 0;

  &:hover {
    background: #0ea5e9;
    transform: scale(1.02);
  }

  &:active {
    transform: scale(0.98);
  }
}

// Command Palette
.sv-command-palette {
  position: absolute;
  top: calc(100% + 12px);
  left: 0;
  right: 0;
  background: var(--sv-card);
  border: 1px solid var(--sv-border);
  border-radius: 12px;
  box-shadow: 0 16px 48px rgba(0, 0, 0, 0.5);
  overflow: hidden;
  animation: slideDown 0.15s ease-out;
  max-height: 400px;
  display: flex;
  flex-direction: column;
  z-index: 100;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-8px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.sv-command-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  border-bottom: 1px solid var(--sv-border);
  color: var(--sv-muted);
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  background: var(--sv-bg-secondary);
}

.sv-command-hint {
  display: flex;
  align-items: center;
  gap: 4px;
  opacity: 0.8;
  font-size: 10px;
}

.sv-command-results {
  overflow-y: auto;
  padding: 8px;
  max-height: 320px;

  &::-webkit-scrollbar {
    width: 8px;
  }

  &::-webkit-scrollbar-track {
    background: var(--sv-bg);
  }

  &::-webkit-scrollbar-thumb {
    background: var(--sv-border);
    border-radius: 4px;

    &:hover {
      background: var(--sv-muted);
    }
  }
}

.sv-command-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.15s ease;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;

  &:hover {
    background: var(--sv-bg-secondary);
  }

  &:not(:last-child) {
    margin-bottom: 4px;
  }
}

.sv-command-icon {
  width: 32px;
  height: 32px;
  border-radius: 6px;
  background: var(--sv-bg-secondary);
  border: 1px solid var(--sv-border);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--sv-accent);
  flex-shrink: 0;
}

.sv-command-content {
  flex: 1;
  text-align: left;
  min-width: 0;
}

.sv-command-title {
  color: var(--sv-text);
  font-weight: 600;
  font-size: 14px;
  margin-bottom: 2px;
}

.sv-command-desc {
  color: var(--sv-muted);
  font-size: 12px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.sv-command-empty {
  padding: 40px 20px;
  text-align: center;
  color: var(--sv-muted);
  font-size: 14px;
}

// Ecosystem
.sv-ecosystem {
  margin-bottom: 40px;
  animation: fadeInUp 0.6s ease-out 0.8s both;

  p {
    color: var(--sv-muted);
    font-size: 13px;
    margin-bottom: 12px;
    font-weight: 500;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  }
}

.sv-icons {
  display: flex;
  gap: 8px;
  justify-content: center;
  flex-wrap: wrap;
}

.sv-ecosystem-badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  background: var(--sv-bg-secondary);
  border: 1px solid var(--sv-border);
  border-radius: 6px;
  color: var(--sv-text);
  font-size: 13px;
  font-weight: 500;
  font-family: 'JetBrains Mono', monospace;
  transition: all 0.15s ease;

  &:hover {
    background: var(--sv-card);
    border-color: var(--sv-muted);
    transform: translateY(-1px);
  }
}

// Social Proof
.sv-social-proof {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  flex-wrap: wrap;
  padding: 16px 24px;
  background: var(--sv-card);
  border: 1px solid var(--sv-border);
  border-radius: 12px;
  animation: fadeInUp 0.6s ease-out 1s both;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
}

.sv-stars {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #FBBF24;
  font-weight: 600;
  font-size: 14px;
}

.sv-divider {
  color: var(--sv-muted);
  font-size: 16px;
}

.sv-featured {
  color: var(--sv-muted);
  font-size: 13px;
  font-weight: 500;

  span {
    color: var(--sv-accent);
    font-weight: 600;
    margin-left: 8px;

    &:not(:last-child)::after {
      content: ',';
      color: var(--sv-muted);
      font-weight: 400;
    }
  }
}

// Animations
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

// Responsive
@media (max-width: 768px) {
  .sv-hero {
    padding: 80px 15px 60px;
  }

  .sv-title {
    font-size: 2.5rem;
  }

  .sv-desc {
    font-size: 1rem;
  }

  .sv-search-box {
    padding: 10px 16px;

    .kdb-shortcut {
      display: none;
    }
  }

  .sv-search-input {
    font-size: 14px;
  }

  .sv-social-proof {
    flex-direction: column;
    gap: 12px;
    font-size: 12px;
  }

  .sv-divider {
    display: none;
  }

  .sv-ecosystem-badge {
    font-size: 11px;
    padding: 5px 10px;
  }

  .sv-command-palette {
    max-height: 300px;
  }
}

@media (max-width: 480px) {
  .sv-title {
    font-size: 2rem;
  }

  .sv-badge {
    font-size: 11px;
    padding: 4px 12px;
  }

  .sv-command-item {
    padding: 10px;
  }
}
</style>
