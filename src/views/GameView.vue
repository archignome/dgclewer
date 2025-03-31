<template>
  <div class="game-container">
    <div v-if="loading" class="loading">
      Loading...
    </div>
    <div v-else>
      <div v-if="!selectedRegion" class="region-selection">
        <h2>Choose Your Path</h2>
        <div class="regions-grid">
          <button
            v-for="region in regions"
            :key="region.id"
            class="region-btn"
            @click="selectRegion(region)"
          >
            <span class="region-name">{{ region.name }}</span>
          </button>
        </div>
      </div>

      <div v-else>
        <div v-if="inCombat" class="combat-view">
          <div class="enemy-info">
            <h3>{{ currentEnemy.name }}</h3>
            <div class="enemy-stats">
              <span>HP: {{ currentEnemy.health }}/{{ currentEnemy.maxHealth }}</span>
            </div>
          </div>
          
          <div class="combat-actions">
            <button @click="attack" class="action-btn">⚔️ Attack</button>
          </div>

          <div class="combat-log">
            {{ combatLog }}
          </div>
        </div>
        
        <div v-else class="region-view">
          <h2>{{ selectedRegion.name }}</h2>
          <p>{{ selectedRegion.description }}</p>
          <button @click="explore" class="explore-btn">
            🔍 Explore
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useGameStore } from '../stores/game'

const router = useRouter()
const gameStore = useGameStore()

const loading = ref(false)
const selectedRegion = ref(null)
const inCombat = ref(false)
const currentEnemy = ref(null)
const combatLog = ref('')

const regions = [
  {
    id: 'forest',
    name: '🌲 Mystic Forest',
    description: 'A mysterious forest filled with magical creatures.',
    enemies: ['Wolf', 'Dark Elf', 'Forest Spirit']
  },
  {
    id: 'cave',
    name: '⛰️ Dragon\'s Cave',
    description: 'A treacherous cave system home to powerful dragons.',
    enemies: ['Young Dragon', 'Cave Troll', 'Dark Knight']
  },
  {
    id: 'castle',
    name: '🏰 Haunted Castle',
    description: 'An ancient castle overrun by undead warriors.',
    enemies: ['Skeleton Knight', 'Ghost', 'Vampire Lord']
  }
]

function selectRegion(region) {
  selectedRegion.value = region
}

function explore() {
  const enemy = generateEnemy()
  currentEnemy.value = enemy
  inCombat.value = true
  combatLog.value = `A ${enemy.name} appears!`
}

function generateEnemy() {
  const enemies = selectedRegion.value.enemies
  const enemyName = enemies[Math.floor(Math.random() * enemies.length)]
  return {
    name: enemyName,
    health: 100,
    maxHealth: 100
  }
}

async function attack() {
  if (!currentEnemy.value) return

  // Player attack
  const damage = Math.floor(Math.random() * 20) + 10
  currentEnemy.value.health -= damage
  combatLog.value = `You hit ${currentEnemy.value.name} for ${damage} damage!`

  // Check if enemy is defeated
  if (currentEnemy.value.health <= 0) {
    const tokens = Math.floor(Math.random() * 20) + 10
    const exp = Math.floor(Math.random() * 30) + 20

    await gameStore.updatePlayerProgress({
      tokens,
      exp,
      stats: {
        battlesWon: 1,
        regionsExplored: 1,
        totalTokens: tokens
      }
    })

    const tg = window.Telegram?.WebApp
    if (tg) {
      tg.showPopup({
        title: 'Victory!',
        message: `You defeated ${currentEnemy.value.name}!\nEarned:\n${tokens} 🪙\n${exp} ✨`
      })
    }

    // Reset combat
    inCombat.value = false
    currentEnemy.value = null
    selectedRegion.value = null
    combatLog.value = ''
    return
  }

  // Enemy attack
  const enemyDamage = Math.floor(Math.random() * 15) + 5
  combatLog.value += `\n${currentEnemy.value.name} hits you for ${enemyDamage} damage!`
}
</script>

<style scoped>
.game-container {
  padding: 1rem;
  min-height: 100vh;
}

.region-selection h2 {
  text-align: center;
  margin-bottom: 2rem;
}

.regions-grid {
  display: grid;
  gap: 1rem;
}

.region-btn {
  padding: 1.5rem;
  border: none;
  border-radius: 12px;
  background: var(--tg-theme-button-color, #2481cc);
  color: var(--tg-theme-button-text-color, #fff);
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.region-btn:hover {
  transform: translateY(-2px);
  filter: brightness(1.1);
}

.region-btn:active {
  transform: translateY(0);
  filter: brightness(0.9);
}

.region-view {
  text-align: center;
  padding: 1rem;
}

.region-view h2 {
  margin-bottom: 1rem;
}

.region-view p {
  margin-bottom: 2rem;
  opacity: 0.8;
}

.explore-btn {
  padding: 1rem 2rem;
  border: none;
  border-radius: 12px;
  background: var(--tg-theme-button-color, #2481cc);
  color: var(--tg-theme-button-text-color, #fff);
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.explore-btn:hover {
  transform: translateY(-2px);
  filter: brightness(1.1);
}

.explore-btn:active {
  transform: translateY(0);
  filter: brightness(0.9);
}

.combat-view {
  padding: 1rem;
  text-align: center;
}

.enemy-info {
  margin-bottom: 2rem;
}

.enemy-info h3 {
  margin-bottom: 0.5rem;
  font-size: 1.5rem;
}

.enemy-stats {
  opacity: 0.8;
}

.combat-actions {
  margin-bottom: 2rem;
}

.action-btn {
  padding: 1rem 2rem;
  border: none;
  border-radius: 12px;
  background: var(--tg-theme-button-color, #2481cc);
  color: var(--tg-theme-button-text-color, #fff);
  font-size: 1.1rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.action-btn:hover {
  transform: translateY(-2px);
  filter: brightness(1.1);
}

.action-btn:active {
  transform: translateY(0);
  filter: brightness(0.9);
}

.combat-log {
  margin-top: 2rem;
  padding: 1rem;
  background: var(--tg-theme-secondary-bg-color, #f5f5f5);
  border-radius: 12px;
  white-space: pre-line;
}
</style>
