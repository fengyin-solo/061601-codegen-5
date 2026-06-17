<script setup lang="ts">
import { ref, computed } from 'vue'
import { useGameStore } from '../stores/gameStore'
import gameConfig from '../config/gameConfig'
import { getRarityColor, getRarityLabel } from '../utils/gameUtils'
import type { AchievementCategory } from '../types/game'

const emit = defineEmits<{
  (e: 'close'): void
}>()

const gameStore = useGameStore()
const selectedCategory = ref<AchievementCategory | 'all'>('all')

const categories: { id: AchievementCategory | 'all'; name: string; icon: string }[] = [
  { id: 'all', name: '全部', icon: '🏆' },
  { id: 'choice', name: '关键选择', icon: '🎯' },
  { id: 'collection', name: '收集进度', icon: '📚' },
  { id: 'ending', name: '结局表现', icon: '🌟' }
]

const filteredAchievements = computed(() => {
  if (selectedCategory.value === 'all') {
    return gameConfig.achievements
  }
  return gameConfig.achievements.filter(a => a.category === selectedCategory.value)
})

const totalAchievements = computed(() => filteredAchievements.value.length)
const unlockedCount = computed(() =>
  filteredAchievements.value.filter(a => gameStore.unlockedAchievements.includes(a.id)).length
)

const overallProgress = computed(() => {
  const total = gameConfig.achievements.length
  const unlocked = gameStore.unlockedAchievements.length
  return total > 0 ? (unlocked / total) * 100 : 0
})

function isUnlocked(achievementId: string): boolean {
  return gameStore.unlockedAchievements.includes(achievementId)
}

function getCategoryLabel(category: AchievementCategory): string {
  const labels: Record<AchievementCategory, string> = {
    choice: '关键选择',
    collection: '收集进度',
    ending: '结局表现'
  }
  return labels[category] || category
}

function getRewardText(reward: { type: string; value: string | number }): string {
  if (reward.type === 'resource') {
    return `+${reward.value} 代币`
  }
  return String(reward.value)
}
</script>

<template>
  <Teleport to="body">
    <div class="modal-overlay" @click.self="emit('close')">
      <div class="modal-content achievement-modal">
        <div class="modal-header">
          <h2>🏆 恋爱成就墙</h2>
          <button class="close-btn" @click="emit('close')">✕</button>
        </div>

        <div class="achievement-overview">
          <div class="overview-stats">
            <div class="stat-item">
              <span class="stat-icon">🏆</span>
              <div class="stat-info">
                <span class="stat-value">{{ gameStore.unlockedAchievements.length }} / {{ gameConfig.achievements.length }}</span>
                <span class="stat-label">已解锁成就</span>
              </div>
            </div>
            <div class="stat-item">
              <span class="stat-icon">📊</span>
              <div class="stat-info">
                <span class="stat-value">{{ overallProgress.toFixed(0) }}%</span>
                <span class="stat-label">完成度</span>
              </div>
            </div>
          </div>
          <div class="overview-progress">
            <div class="progress-bar">
              <div 
                class="progress-fill" 
                :style="{ width: `${overallProgress}%` }"
              ></div>
            </div>
          </div>
        </div>

        <div class="category-tabs">
          <button 
            v-for="cat in categories" 
            :key="cat.id"
            class="tab-btn"
            :class="{ active: selectedCategory === cat.id }"
            @click="selectedCategory = cat.id"
          >
            <span class="tab-icon">{{ cat.icon }}</span>
            <span>{{ cat.name }}</span>
          </button>
        </div>

        <div class="achievements-grid">
          <div
            v-for="achievement in filteredAchievements"
            :key="achievement.id"
            class="achievement-card"
            :class="{ 
              unlocked: isUnlocked(achievement.id), 
              locked: !isUnlocked(achievement.id)
            }"
          >
            <div 
              class="achievement-icon-wrapper"
              :style="{ borderColor: getRarityColor(achievement.rarity) }"
            >
              <span v-if="isUnlocked(achievement.id)" class="achievement-icon">
                {{ achievement.icon }}
              </span>
              <span v-else class="achievement-locked">🔒</span>
            </div>
            
            <div class="achievement-info">
              <div class="achievement-header">
                <span class="achievement-name">
                  {{ isUnlocked(achievement.id) ? achievement.name : '???' }}
                </span>
                <span 
                  class="achievement-rarity badge" 
                  :class="`badge-${achievement.rarity}`"
                >
                  {{ getRarityLabel(achievement.rarity) }}
                </span>
              </div>
              
              <span class="achievement-category">
                {{ getCategoryLabel(achievement.category) }}
              </span>
              
              <p class="achievement-desc" :class="{ locked: !isUnlocked(achievement.id) }">
                {{ isUnlocked(achievement.id) ? achievement.description : '未解锁' }}
              </p>
              
              <div v-if="achievement.reward && isUnlocked(achievement.id)" class="achievement-reward">
                <span class="reward-label">奖励:</span>
                <span class="reward-value">{{ getRewardText(achievement.reward) }}</span>
              </div>
              <div v-else-if="achievement.reward && !isUnlocked(achievement.id)" class="achievement-reward locked">
                <span class="reward-label">奖励:</span>
                <span class="reward-value">???</span>
              </div>
            </div>
          </div>
        </div>

        <div class="achievement-footer">
          <span class="footer-text">
            本分类已解锁 <strong>{{ unlockedCount }}</strong> / {{ totalAchievements }} 个成就
          </span>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.achievement-modal {
  padding: 24px;
  max-width: 800px;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.modal-header h2 {
  font-size: 20px;
  font-weight: 600;
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.close-btn {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: var(--bg-tertiary);
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.close-btn:hover {
  background: var(--accent-light);
}

.achievement-overview {
  background: var(--bg-tertiary);
  border-radius: var(--radius-lg);
  padding: 20px;
  margin-bottom: 20px;
}

.overview-stats {
  display: flex;
  gap: 32px;
  margin-bottom: 16px;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 12px;
}

.stat-icon {
  font-size: 32px;
}

.stat-info {
  display: flex;
  flex-direction: column;
}

.stat-value {
  font-size: 20px;
  font-weight: 700;
  color: var(--accent-primary);
}

.stat-label {
  font-size: 12px;
  color: var(--text-secondary);
}

.overview-progress .progress-bar {
  height: 10px;
  background: var(--bg-secondary);
}

.overview-progress .progress-fill {
  background: linear-gradient(90deg, var(--accent-primary), var(--accent-secondary));
  height: 100%;
  border-radius: 5px;
  transition: width 0.3s;
}

.category-tabs {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 20px;
  padding-bottom: 16px;
  border-bottom: 1px solid var(--border-light);
}

.tab-btn {
  padding: 10px 16px;
  border-radius: var(--radius-md);
  background: var(--bg-tertiary);
  font-size: 13px;
  font-weight: 500;
  color: var(--text-secondary);
  transition: all 0.2s;
  display: flex;
  align-items: center;
  gap: 6px;
}

.tab-btn:hover {
  background: var(--accent-light);
}

.tab-btn.active {
  background: linear-gradient(135deg, var(--accent-primary), var(--accent-secondary));
  color: white;
}

.tab-icon {
  font-size: 16px;
}

.achievements-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 14px;
  max-height: 400px;
  overflow-y: auto;
  padding-right: 8px;
}

.achievement-card {
  display: flex;
  gap: 14px;
  padding: 16px;
  background: var(--bg-tertiary);
  border-radius: var(--radius-lg);
  transition: all 0.2s;
  border: 2px solid transparent;
}

.achievement-card.unlocked {
  background: var(--accent-light);
  border-color: var(--accent-primary);
}

.achievement-card.locked {
  opacity: 0.6;
}

.achievement-icon-wrapper {
  width: 64px;
  height: 64px;
  border-radius: var(--radius-md);
  background: var(--bg-secondary);
  display: flex;
  align-items: center;
  justify-content: center;
  border: 3px solid var(--border-color);
  flex-shrink: 0;
}

.achievement-icon {
  font-size: 32px;
}

.achievement-locked {
  font-size: 24px;
  opacity: 0.5;
}

.achievement-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 0;
}

.achievement-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.achievement-name {
  font-weight: 600;
  font-size: 14px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.achievement-rarity {
  font-size: 10px;
  flex-shrink: 0;
}

.achievement-category {
  font-size: 11px;
  color: var(--accent-primary);
  font-weight: 500;
}

.achievement-desc {
  font-size: 12px;
  color: var(--text-secondary);
  line-height: 1.4;
  margin-top: 2px;
}

.achievement-desc.locked {
  color: var(--text-muted);
  font-style: italic;
}

.achievement-reward {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-top: 6px;
  font-size: 11px;
}

.achievement-reward .reward-label {
  color: var(--text-secondary);
}

.achievement-reward .reward-value {
  color: var(--success-color);
  font-weight: 600;
}

.achievement-reward.locked .reward-value {
  color: var(--text-muted);
}

.achievement-footer {
  margin-top: 16px;
  padding-top: 12px;
  border-top: 1px solid var(--border-light);
  text-align: center;
}

.footer-text {
  font-size: 13px;
  color: var(--text-secondary);
}

.footer-text strong {
  color: var(--accent-primary);
  font-size: 15px;
}

@media (max-width: 600px) {
  .achievements-grid {
    grid-template-columns: 1fr;
  }
  
  .overview-stats {
    flex-direction: column;
    gap: 12px;
  }
}
</style>
