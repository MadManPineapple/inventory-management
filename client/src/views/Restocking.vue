<script setup>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'

const loading = ref(true)
const error = ref(null)
const forecasts = ref([])
const budget = ref(50000)
const orderSubmitted = ref(false)
const submitting = ref(false)
const submittedOrder = ref(null)

const recommendations = computed(() => {
  const sorted = [...forecasts.value].sort((a, b) => b.forecasted_demand - a.forecasted_demand)
  let remaining = budget.value
  const result = []
  for (const item of sorted) {
    if (remaining <= 0) break
    if (!item.unit_cost || item.unit_cost <= 0) continue
    const maxAffordable = Math.floor(remaining / item.unit_cost)
    if (maxAffordable <= 0) continue
    const quantity = Math.min(item.forecasted_demand, maxAffordable)
    const line_total = Math.round(quantity * item.unit_cost * 100) / 100
    result.push({ sku: item.item_sku, name: item.item_name, forecasted_demand: item.forecasted_demand, quantity, unit_cost: item.unit_cost, line_total })
    remaining -= line_total
  }
  return result
})

const totalCost = computed(() =>
  Math.round(recommendations.value.reduce((sum, item) => sum + item.line_total, 0) * 100) / 100
)

const budgetRemaining = computed(() => Math.round((budget.value - totalCost.value) * 100) / 100)

const loadForecasts = async () => {
  try {
    loading.value = true
    error.value = null
    forecasts.value = await api.getDemandForecasts()
  } catch (err) {
    error.value = 'Failed to load demand forecasts: ' + err.message
  } finally {
    loading.value = false
  }
}

const placeOrder = async () => {
  if (recommendations.value.length === 0 || submitting.value || orderSubmitted.value) return
  try {
    submitting.value = true
    error.value = null
    submittedOrder.value = await api.submitRestockingOrder(recommendations.value, totalCost.value)
    orderSubmitted.value = true
  } catch (err) {
    error.value = 'Failed to submit restocking order: ' + err.message
  } finally {
    submitting.value = false
  }
}

const formatDate = (dateString) => {
  return new Date(dateString).toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
  })
}

onMounted(loadForecasts)
</script>

<template>
  <div>
    <div class="page-header">
      <h2>Restocking</h2>
      <p>Set your available budget to get restocking recommendations based on demand forecasts.</p>
    </div>

    <div v-if="loading" class="loading">Loading demand forecasts...</div>

    <div v-else-if="error" class="error">{{ error }}</div>

    <div v-else>
      <div v-if="orderSubmitted && submittedOrder" class="success-banner">
        <h4>Order Submitted Successfully</h4>
        <p>Order <strong>{{ submittedOrder.order_number }}</strong> has been placed.</p>
        <p>Submitted: {{ formatDate(submittedOrder.submitted_date) }} &mdash; Expected delivery: {{ formatDate(submittedOrder.expected_delivery) }}</p>
      </div>

      <div class="card">
        <div class="card-header">
          <h3 class="card-title">Available Budget</h3>
        </div>
        <div style="padding: 1rem 1.25rem 1.25rem;">
          <div class="budget-slider-row">
            <input
              type="range"
              min="1000"
              max="500000"
              step="1000"
              v-model.number="budget"
              class="budget-slider"
            />
            <span class="budget-value">${{ budget.toLocaleString() }}</span>
          </div>
          <p class="budget-meta">
            Budget remaining after recommendations:
            <strong>${{ budgetRemaining.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</strong>
          </p>
        </div>
      </div>

      <div class="card">
        <div class="card-header">
          <h3 class="card-title">Recommended Restocking Items ({{ recommendations.length }})</h3>
        </div>

        <div v-if="recommendations.length === 0" class="empty-state">
          No items can be recommended within the current budget. Try increasing the budget.
        </div>

        <div v-else class="table-container">
          <table>
            <thead>
              <tr>
                <th>SKU</th>
                <th>Item Name</th>
                <th>Forecasted Demand</th>
                <th>Qty to Order</th>
                <th>Unit Cost</th>
                <th>Line Total</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendations" :key="item.sku">
                <td>{{ item.sku }}</td>
                <td>{{ item.name }}</td>
                <td>{{ item.forecasted_demand.toLocaleString() }}</td>
                <td>{{ item.quantity.toLocaleString() }}</td>
                <td>${{ item.unit_cost.toLocaleString() }}</td>
                <td>${{ item.line_total.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div v-if="recommendations.length > 0" class="order-summary">
          <span class="total-label">
            Total Cost:
            <strong class="total-value">${{ totalCost.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</strong>
            of ${{ budget.toLocaleString() }} budget
          </span>
          <button
            class="btn-place-order"
            :disabled="orderSubmitted || submitting || recommendations.length === 0"
            @click="placeOrder"
          >
            <span v-if="submitting">Submitting...</span>
            <span v-else-if="orderSubmitted">Order Submitted</span>
            <span v-else>Place Order</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.budget-slider-row {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  margin: 1rem 0;
}

.budget-slider {
  flex: 1;
  accent-color: #2563eb;
  cursor: pointer;
}

.budget-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
  min-width: 130px;
}

.budget-meta {
  font-size: 0.875rem;
  color: #64748b;
}

.order-summary {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0.75rem;
  border-top: 2px solid #e2e8f0;
  margin-top: 0.5rem;
}

.total-label {
  font-size: 0.875rem;
  color: #64748b;
  font-weight: 600;
}

.total-value {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
}

.btn-place-order {
  padding: 0.75rem 2rem;
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.938rem;
  cursor: pointer;
  transition: background 0.2s;
}

.btn-place-order:hover:not(:disabled) {
  background: #1d4ed8;
}

.btn-place-order:disabled {
  background: #94a3b8;
  cursor: not-allowed;
}

.success-banner {
  background: #d1fae5;
  border: 1px solid #6ee7b7;
  border-radius: 8px;
  padding: 1.25rem;
  margin-bottom: 1.25rem;
  color: #065f46;
}

.success-banner h4 {
  font-size: 1rem;
  font-weight: 700;
  margin-bottom: 0.5rem;
}

.success-banner p {
  margin: 0.25rem 0;
  font-size: 0.875rem;
}

.empty-state {
  padding: 2rem;
  text-align: center;
  color: #64748b;
}
</style>
