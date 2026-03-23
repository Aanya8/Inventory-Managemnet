<template>
  <div class="restocking">
    <div class="page-header">
      <h2>Restocking</h2>
      <p>Set your available budget and review recommended items to restock.</p>
    </div>

    <!-- Budget Section -->
    <div class="card budget-card">
      <div class="card-header">
        <h3 class="card-title">Available Budget</h3>
        <span class="budget-display">{{ currencySymbol }}{{ budget.toLocaleString() }}</span>
      </div>
      <input
        type="range"
        class="budget-slider"
        min="0"
        max="50000"
        step="500"
        v-model.number="budget"
      />
      <div class="budget-range-labels">
        <span>{{ currencySymbol }}0</span>
        <span>{{ currencySymbol }}50,000</span>
      </div>
    </div>

    <!-- Budget summary bar -->
    <div v-if="recommendations.length > 0 && !loading" class="budget-summary-bar">
      <span>{{ itemsCount }} item{{ itemsCount !== 1 ? 's' : '' }} recommended</span>
      <span class="separator">·</span>
      <span>{{ currencySymbol }}{{ totalCost.toLocaleString() }} total cost</span>
      <span class="separator">·</span>
      <span :class="remainingBudget >= 0 ? 'remaining-positive' : 'remaining-negative'">
        {{ currencySymbol }}{{ Math.abs(remainingBudget).toLocaleString() }} remaining
      </span>
    </div>

    <!-- Loading state -->
    <div v-if="loading" class="loading">Loading recommendations...</div>

    <!-- Error state -->
    <div v-else-if="error" class="error">{{ error }}</div>

    <!-- Empty state -->
    <div v-else-if="recommendations.length === 0" class="card empty-state">
      <p>No items need restocking within the current budget, or all items are sufficiently stocked.</p>
    </div>

    <!-- Recommendations table -->
    <div v-else class="card">
      <div class="card-header">
        <h3 class="card-title">Recommended Items ({{ recommendations.length }})</h3>
        <button
          class="place-order-btn"
          :disabled="submitting || !!successOrder || recommendations.length === 0"
          @click="placeOrder"
        >
          {{ submitting ? 'Placing Order...' : 'Place Order' }}
        </button>
      </div>
      <div class="table-container">
        <table>
          <thead>
            <tr>
              <th>SKU</th>
              <th>Item Name</th>
              <th>Category</th>
              <th>Warehouse</th>
              <th>On Hand</th>
              <th>Reorder Point</th>
              <th>Qty to Order</th>
              <th>Unit Cost</th>
              <th>Total Cost</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in recommendations" :key="item.id">
              <td><strong>{{ item.sku }}</strong></td>
              <td>{{ item.name }}</td>
              <td>{{ item.category }}</td>
              <td>{{ item.warehouse }}</td>
              <td>{{ item.quantity_on_hand.toLocaleString() }}</td>
              <td>{{ item.reorder_point.toLocaleString() }}</td>
              <td><strong>{{ item.quantity_to_order.toLocaleString() }}</strong></td>
              <td>{{ currencySymbol }}{{ item.unit_cost.toLocaleString() }}</td>
              <td><strong>{{ currencySymbol }}{{ item.total_cost.toLocaleString() }}</strong></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Success banner -->
    <div v-if="successOrder" class="success-banner">
      Order <strong>{{ successOrder.order_number }}</strong> placed successfully.
      Expected delivery: {{ successOrder.expected_delivery }}
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
    const { currentCurrency } = useI18n()

    const currencySymbol = computed(() => {
      return currentCurrency.value === 'JPY' ? '¥' : '$'
    })

    const budget = ref(10000)
    const recommendations = ref([])
    const totalCost = ref(0)
    const remainingBudget = ref(0)
    const itemsCount = ref(0)
    const loading = ref(false)
    const error = ref(null)
    const submitting = ref(false)
    const successOrder = ref(null)
    const debounceTimer = ref(null)

    const loadRecommendations = async () => {
      loading.value = true
      error.value = null
      try {
        const data = await api.getRestockingRecommendations(budget.value)
        recommendations.value = data.recommendations
        totalCost.value = data.total_cost
        remainingBudget.value = data.remaining_budget
        itemsCount.value = data.items_count
      } catch (err) {
        error.value = 'Failed to load restocking recommendations'
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    const placeOrder = async () => {
      submitting.value = true
      error.value = null
      try {
        const order = await api.submitRestockingOrder(recommendations.value, budget.value)
        successOrder.value = order
        recommendations.value = []
        totalCost.value = 0
        remainingBudget.value = budget.value
        itemsCount.value = 0
      } catch (err) {
        error.value = 'Failed to place restocking order'
        console.error(err)
      } finally {
        submitting.value = false
      }
    }

    watch(budget, () => {
      if (debounceTimer.value) {
        clearTimeout(debounceTimer.value)
      }
      successOrder.value = null
      debounceTimer.value = setTimeout(() => {
        loadRecommendations()
      }, 300)
    })

    onMounted(() => loadRecommendations())

    return {
      currencySymbol,
      budget,
      recommendations,
      totalCost,
      remainingBudget,
      itemsCount,
      loading,
      error,
      submitting,
      successOrder,
      loadRecommendations,
      placeOrder
    }
  }
}
</script>

<style scoped>
.restocking {
  padding: 0;
}

.budget-card .card-header {
  margin-bottom: 1rem;
}

.budget-display {
  font-size: 1.5rem;
  font-weight: 700;
  color: #2563eb;
  letter-spacing: -0.025em;
}

.budget-slider {
  width: 100%;
  height: 6px;
  appearance: none;
  -webkit-appearance: none;
  background: #e2e8f0;
  border-radius: 3px;
  outline: none;
  cursor: pointer;
  accent-color: #2563eb;
}

.budget-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #2563eb;
  cursor: pointer;
  border: 2px solid white;
  box-shadow: 0 1px 4px rgba(37, 99, 235, 0.4);
}

.budget-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #2563eb;
  cursor: pointer;
  border: 2px solid white;
  box-shadow: 0 1px 4px rgba(37, 99, 235, 0.4);
}

.budget-range-labels {
  display: flex;
  justify-content: space-between;
  margin-top: 0.5rem;
  font-size: 0.813rem;
  color: #64748b;
}

.budget-summary-bar {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 0.75rem 1.25rem;
  margin-bottom: 1.25rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.875rem;
  font-weight: 500;
  color: #0f172a;
}

.separator {
  color: #cbd5e1;
}

.remaining-positive {
  color: #059669;
}

.remaining-negative {
  color: #dc2626;
}

.empty-state {
  text-align: center;
  padding: 2rem;
  color: #64748b;
  font-size: 0.938rem;
}

.place-order-btn {
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 6px;
  padding: 0.5rem 1.25rem;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.2s ease;
}

.place-order-btn:hover:not(:disabled) {
  background: #1d4ed8;
}

.place-order-btn:disabled {
  background: #93c5fd;
  cursor: not-allowed;
}

.success-banner {
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  color: #065f46;
  border-radius: 8px;
  padding: 1rem 1.25rem;
  margin-top: 1.25rem;
  font-size: 0.938rem;
}
</style>
