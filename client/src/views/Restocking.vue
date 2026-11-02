<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <!-- Budget Slider Card -->
      <div class="card budget-card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.budget.title') }}</h3>
        </div>
        <div class="budget-slider-container">
          <input
            type="range"
            :min="1000"
            :max="50000"
            :step="1000"
            v-model.number="budget"
            class="budget-slider"
          />
          <div class="budget-display">
            <span class="budget-value">{{ currencySymbol }}{{ budget.toLocaleString() }}</span>
            <span class="budget-remaining">
              {{ t('restocking.budget.remaining') }}: {{ currencySymbol }}{{ remainingBudget.toLocaleString() }}
            </span>
          </div>
        </div>
      </div>

      <!-- Stats Grid -->
      <div class="stats-grid">
        <div class="stat-card info">
          <div class="stat-label">{{ t('restocking.stats.selectedItems') }}</div>
          <div class="stat-value">{{ selectedItems.length }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">{{ t('restocking.stats.totalCost') }}</div>
          <div class="stat-value">{{ currencySymbol }}{{ totalCost.toLocaleString() }}</div>
        </div>
        <div class="stat-card success">
          <div class="stat-label">{{ t('restocking.stats.budgetRemaining') }}</div>
          <div class="stat-value">{{ currencySymbol }}{{ remainingBudget.toLocaleString() }}</div>
        </div>
      </div>

      <!-- Recommendations Table -->
      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.recommendations.title') }}</h3>
          <button
            class="place-order-btn"
            :disabled="selectedItems.length === 0 || submitting"
            @click="placeOrder"
          >
            {{ submitting ? t('common.loading') : t('restocking.placeOrder') }}
          </button>
        </div>
        <div class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('restocking.table.select') }}</th>
                <th>{{ t('restocking.table.sku') }}</th>
                <th>{{ t('restocking.table.itemName') }}</th>
                <th>{{ t('restocking.table.forecastedDemand') }}</th>
                <th>{{ t('restocking.table.trend') }}</th>
                <th>{{ t('restocking.table.unitCost') }}</th>
                <th>{{ t('restocking.table.quantity') }}</th>
                <th>{{ t('restocking.table.subtotal') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendations" :key="item.id">
                <td>
                  <input
                    type="checkbox"
                    :checked="isSelected(item)"
                    :disabled="!canSelect(item)"
                    @change="toggleSelection(item)"
                  />
                </td>
                <td><strong>{{ item.item_sku }}</strong></td>
                <td>{{ item.item_name }}</td>
                <td><strong>{{ item.forecasted_demand }}</strong></td>
                <td>
                  <span :class="['badge', item.trend]">
                    {{ t(`trends.${item.trend}`) }}
                  </span>
                </td>
                <td>{{ currencySymbol }}{{ item.unit_cost.toFixed(2) }}</td>
                <td>
                  <input
                    type="number"
                    :value="getQuantity(item)"
                    :min="1"
                    :max="getMaxQuantity(item)"
                    :disabled="!isSelected(item)"
                    @input="updateQuantity(item, $event.target.value)"
                    class="quantity-input"
                  />
                </td>
                <td>
                  <strong>{{ currencySymbol }}{{ getSubtotal(item).toLocaleString() }}</strong>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t, currentCurrency } = useI18n()

    const currencySymbol = computed(() => currentCurrency.value === 'JPY' ? '¥' : '$')
    const loading = ref(true)
    const error = ref(null)
    const submitting = ref(false)
    const budget = ref(10000)
    const recommendations = ref([])
    const selectedItems = ref([])

    const loadRecommendations = async () => {
      try {
        loading.value = true
        error.value = null
        const data = await api.getRestockingRecommendations(budget.value)
        recommendations.value = data
      } catch (err) {
        error.value = 'Failed to load recommendations: ' + err.message
      } finally {
        loading.value = false
      }
    }

    const totalCost = computed(() => {
      return selectedItems.value.reduce((sum, sel) => sum + sel.subtotal, 0)
    })

    const remainingBudget = computed(() => {
      return Math.max(0, budget.value - totalCost.value)
    })

    const isSelected = (item) => {
      return selectedItems.value.some(s => s.item_sku === item.item_sku)
    }

    const canSelect = (item) => {
      if (isSelected(item)) return true
      return item.unit_cost <= remainingBudget.value
    }

    const getQuantity = (item) => {
      const sel = selectedItems.value.find(s => s.item_sku === item.item_sku)
      return sel ? sel.quantity : 1
    }

    const getMaxQuantity = (item) => {
      const sel = selectedItems.value.find(s => s.item_sku === item.item_sku)
      const currentSubtotal = sel ? sel.subtotal : 0
      const availableBudget = remainingBudget.value + currentSubtotal
      return Math.max(1, Math.floor(availableBudget / item.unit_cost))
    }

    const getSubtotal = (item) => {
      const sel = selectedItems.value.find(s => s.item_sku === item.item_sku)
      return sel ? sel.subtotal : 0
    }

    const toggleSelection = (item) => {
      const idx = selectedItems.value.findIndex(s => s.item_sku === item.item_sku)
      if (idx >= 0) {
        selectedItems.value.splice(idx, 1)
      } else {
        const maxQty = getMaxQuantity(item)
        const qty = Math.min(item.forecasted_demand, maxQty)
        selectedItems.value.push({
          item_sku: item.item_sku,
          item_name: item.item_name,
          unit_cost: item.unit_cost,
          quantity: qty,
          subtotal: qty * item.unit_cost
        })
      }
    }

    const updateQuantity = (item, newQty) => {
      const sel = selectedItems.value.find(s => s.item_sku === item.item_sku)
      if (sel) {
        const maxQty = getMaxQuantity(item)
        const qty = Math.max(1, Math.min(parseInt(newQty) || 1, maxQty))
        sel.quantity = qty
        sel.subtotal = qty * item.unit_cost
      }
    }

    const placeOrder = async () => {
      if (selectedItems.value.length === 0) return

      try {
        submitting.value = true
        error.value = null
        const orderData = {
          budget: budget.value,
          items: selectedItems.value,
          total_value: totalCost.value
        }

        await api.createRestockingOrder(orderData)

        selectedItems.value = []
        alert(t('restocking.orderSuccess'))
      } catch (err) {
        error.value = 'Failed to place order: ' + err.message
      } finally {
        submitting.value = false
      }
    }

    // Reload recommendations when budget changes (debounced via watch)
    let budgetTimeout = null
    watch(budget, () => {
      clearTimeout(budgetTimeout)
      budgetTimeout = setTimeout(() => {
        loadRecommendations()
      }, 300)
    })

    onMounted(loadRecommendations)

    return {
      t,
      currencySymbol,
      loading,
      error,
      submitting,
      budget,
      recommendations,
      selectedItems,
      totalCost,
      remainingBudget,
      isSelected,
      canSelect,
      getQuantity,
      getMaxQuantity,
      getSubtotal,
      toggleSelection,
      updateQuantity,
      placeOrder
    }
  }
}
</script>

<style scoped>
.budget-card {
  margin-bottom: 1.5rem;
}

.budget-slider-container {
  padding: 1rem 0;
}

.budget-slider {
  width: 100%;
  height: 8px;
  border-radius: 4px;
  background: #e2e8f0;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
}

.budget-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(59, 130, 246, 0.4);
  transition: transform 0.2s;
}

.budget-slider::-webkit-slider-thumb:hover {
  transform: scale(1.1);
}

.budget-slider::-moz-range-thumb {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: #3b82f6;
  cursor: pointer;
  border: none;
  box-shadow: 0 2px 6px rgba(59, 130, 246, 0.4);
}

.budget-display {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 1rem;
}

.budget-value {
  font-size: 2rem;
  font-weight: 700;
  color: #0f172a;
}

.budget-remaining {
  font-size: 1rem;
  color: #64748b;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 1.25rem;
  text-align: center;
}

.stat-card.info {
  border-left: 4px solid #3b82f6;
}

.stat-card.success {
  border-left: 4px solid #10b981;
}

.stat-label {
  font-size: 0.875rem;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.stat-value {
  font-size: 1.75rem;
  font-weight: 700;
  color: #0f172a;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.place-order-btn {
  background: #3b82f6;
  color: white;
  border: none;
  padding: 0.75rem 1.5rem;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.place-order-btn:hover:not(:disabled) {
  background: #2563eb;
  transform: translateY(-1px);
}

.place-order-btn:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.quantity-input {
  width: 80px;
  padding: 0.5rem;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  text-align: center;
  font-size: 0.875rem;
}

.quantity-input:disabled {
  background: #f1f5f9;
  color: #94a3b8;
}

.quantity-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

input[type="checkbox"] {
  width: 18px;
  height: 18px;
  cursor: pointer;
}

input[type="checkbox"]:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

.badge {
  display: inline-block;
  padding: 0.25rem 0.75rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: capitalize;
}

.badge.increasing {
  background: #d1fae5;
  color: #059669;
}

.badge.stable {
  background: #dbeafe;
  color: #2563eb;
}

.badge.decreasing {
  background: #fee2e2;
  color: #dc2626;
}
</style>
