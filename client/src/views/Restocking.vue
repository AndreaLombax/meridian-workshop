<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div class="budget-bar">
      <label class="budget-label">{{ t('restocking.budgetCeiling') }}</label>
      <div class="budget-input-wrapper">
        <span class="currency-prefix">{{ currencySymbol }}</span>
        <input
          type="number"
          v-model.number="budgetInput"
          min="0"
          step="1000"
          @input="onBudgetInput"
          placeholder="0"
        />
      </div>
    </div>

    <div class="stats-grid" v-if="!loading && recommendations.length">
      <div class="stat-card">
        <div class="stat-label">{{ t('restocking.itemsRecommended') }}</div>
        <div class="stat-value">{{ withinBudgetItems.length }} <span class="stat-total">/ {{ recommendations.length }}</span></div>
      </div>
      <div class="stat-card">
        <div class="stat-label">{{ t('restocking.totalCost') }}</div>
        <div class="stat-value">{{ currencySymbol }}{{ formatNumber(totalCost) }}</div>
      </div>
      <div class="stat-card" v-if="budget > 0">
        <div class="stat-label">{{ t('restocking.budgetRemaining') }}</div>
        <div class="stat-value" :class="budgetRemaining < 0 ? 'text-danger' : ''">
          {{ currencySymbol }}{{ formatNumber(budgetRemaining) }}
        </div>
      </div>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else-if="recommendations.length === 0" class="empty-state">
      All items are sufficiently stocked. No restocking needed.
    </div>
    <div v-else class="card">
      <div class="card-header">
        <h3 class="card-title">Recommendations ({{ recommendations.length }})</h3>
      </div>
      <div class="table-container">
        <table class="restocking-table">
          <thead>
            <tr>
              <th>Item</th>
              <th>Warehouse</th>
              <th>Stock</th>
              <th>Forecast</th>
              <th>To Order</th>
              <th>Unit Cost</th>
              <th>Est. Cost</th>
              <th>Urgency</th>
              <th>Budget</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="r in recommendations"
              :key="r.sku"
              :class="{ 'row-over-budget': !r.within_budget }"
            >
              <td>
                <div class="item-name">{{ r.name }}</div>
                <div class="item-sku">{{ r.sku }}</div>
              </td>
              <td>{{ r.warehouse }}</td>
              <td>{{ r.quantity_on_hand }}</td>
              <td>{{ r.forecasted_demand ?? '—' }}</td>
              <td><strong>{{ r.quantity_to_order }}</strong></td>
              <td>{{ currencySymbol }}{{ formatNumber(r.unit_cost) }}</td>
              <td>{{ currencySymbol }}{{ formatNumber(r.estimated_cost) }}</td>
              <td><span :class="urgencyClass(r.urgency)">{{ r.urgency }}</span></td>
              <td>
                <span :class="r.within_budget ? 'badge success' : 'badge danger'">
                  {{ r.within_budget ? t('restocking.withinBudget') : t('restocking.overBudget') }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'
import { api } from '../api'
import { useFilters } from '../composables/useFilters'
import { useI18n } from '../composables/useI18n'

export default {
  name: 'Restocking',
  setup() {
    const { t, currentCurrency } = useI18n()
    const { selectedLocation, selectedCategory, getCurrentFilters } = useFilters()

    const currencySymbol = computed(() => currentCurrency.value === 'JPY' ? '¥' : '$')

    const budgetInput = ref(0)
    const budget = ref(0)
    const recommendations = ref([])
    const loading = ref(false)
    const error = ref(null)
    let debounceTimer = null

    const withinBudgetItems = computed(() => recommendations.value.filter(r => r.within_budget))
    const totalCost = computed(() => withinBudgetItems.value.reduce((s, r) => s + r.estimated_cost, 0))
    const budgetRemaining = computed(() => Math.max(0, budget.value - totalCost.value))

    const loadData = async () => {
      try {
        loading.value = true
        error.value = null
        const filters = { ...getCurrentFilters(), budget: budget.value }
        recommendations.value = await api.getRestockingRecommendations(filters)
      } catch (err) {
        error.value = 'Failed to load recommendations: ' + err.message
      } finally {
        loading.value = false
      }
    }

    const onBudgetInput = () => {
      clearTimeout(debounceTimer)
      debounceTimer = setTimeout(() => {
        budget.value = budgetInput.value
        loadData()
      }, 400)
    }

    const urgencyClass = (urgency) => {
      return { critical: 'badge danger', high: 'badge warning', medium: 'badge info' }[urgency] || 'badge'
    }

    const formatNumber = (n) =>
      Number(n).toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 })

    watch([selectedLocation, selectedCategory], loadData)
    onMounted(loadData)

    return {
      t, currencySymbol, budgetInput, budget, recommendations, loading, error,
      withinBudgetItems, totalCost, budgetRemaining,
      onBudgetInput, urgencyClass, formatNumber
    }
  }
}
</script>

<style scoped>
.restocking { padding: 0; }

.budget-bar {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.budget-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #374151;
}

.budget-input-wrapper {
  display: flex;
  align-items: center;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  overflow: hidden;
  background: white;
}

.currency-prefix {
  padding: 0.5rem 0.75rem;
  background: #f8fafc;
  color: #64748b;
  font-weight: 600;
  border-right: 1px solid #e2e8f0;
}

input[type=number] {
  border: none;
  outline: none;
  padding: 0.5rem 0.75rem;
  font-size: 1rem;
  width: 180px;
  color: #0f172a;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  border-radius: 12px;
  padding: 1.25rem 1.5rem;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  border-left: 4px solid #3b82f6;
}

.stat-label {
  font-size: 0.875rem;
  color: #64748b;
  margin-bottom: 0.4rem;
}

.stat-value {
  font-size: 1.75rem;
  font-weight: 700;
  color: #0f172a;
}

.stat-total {
  font-size: 1rem;
  font-weight: 400;
  color: #94a3b8;
}

.text-danger { color: #dc2626; }

.card {
  background: white;
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.card-header { margin-bottom: 1rem; }

.card-title {
  font-size: 1.1rem;
  font-weight: 600;
  color: #0f172a;
}

.table-container { overflow-x: auto; }

.restocking-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}

.restocking-table th {
  background: #f8fafc;
  padding: 0.75rem;
  text-align: left;
  font-weight: 600;
  color: #64748b;
  border-bottom: 2px solid #e2e8f0;
  white-space: nowrap;
}

.restocking-table td {
  padding: 0.75rem;
  border-bottom: 1px solid #f1f5f9;
  vertical-align: middle;
}

.restocking-table tr:hover td { background: #f8fafc; }

.row-over-budget td { opacity: 0.5; }

.item-name { font-weight: 500; color: #0f172a; }
.item-sku { font-size: 0.75rem; color: #94a3b8; font-family: monospace; margin-top: 2px; }

.badge {
  padding: 0.2rem 0.6rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: capitalize;
  display: inline-block;
}

.badge.danger  { background: #fee2e2; color: #991b1b; }
.badge.warning { background: #fef3c7; color: #92400e; }
.badge.info    { background: #e0f2fe; color: #0369a1; }
.badge.success { background: #dcfce7; color: #166534; }

.empty-state {
  text-align: center;
  padding: 3rem;
  color: #64748b;
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.loading {
  text-align: center;
  padding: 3rem;
  color: #64748b;
}

.error {
  background: #fee2e2;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
}
</style>
