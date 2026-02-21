<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import type { BuyerProfile, Order, OrderStatus } from "../types";
import api from "../services/api";
import { fetchOrders } from "../services/orders";

const profile = ref<BuyerProfile | null>(null);
const orders = ref<Order[]>([]);
const loading = ref(true);

onMounted(async () => {
  const [profileRes, ordersData] = await Promise.all([
    api.get<BuyerProfile>("/api/buyers/me/"),
    fetchOrders(),
  ]);
  profile.value = profileRes.data;
  orders.value = ordersData;
  loading.value = false;
});

const statusCounts = computed<Record<OrderStatus, number>>(() => {
  const counts: Record<string, number> = {};
  for (const order of orders.value) {
    counts[order.status] = (counts[order.status] ?? 0) + 1;
  }
  return counts as Record<OrderStatus, number>;
});

const recentOrders = computed(() =>
  [...orders.value].sort((a, b) => b.id - a.id).slice(0, 3)
);

function formatDate(iso: string): string {
  return new Date(iso).toLocaleDateString("en-GB", {
    day: "numeric",
    month: "short",
    year: "numeric",
  });
}

const ALL_STATUSES: OrderStatus[] = [
  "PENDING",
  "CONFIRMED",
  "PROCESSING",
  "SHIPPED",
  "DELIVERED",
  "CANCELLED",
];
</script>

<template>
  <div v-if="loading" class="loading">Loading…</div>
  <div v-else class="dashboard">
    <!-- Challenge Overview -->
    <section class="challenge-overview">
      <h2>Challenge Overview</h2>
      <p>
        <strong>Welcome to Opply's engineering code challenge.</strong> Opply is a B2B procurement
        platform that connects <em>Buyers</em> (food &amp; beverage brands) with <em>Suppliers</em>
        (ingredient manufacturers). Opply acts as a merchant of record — buyers order through the
        platform and Opply manages supplier relationships, pricing, and fulfilment.
      </p>
      <h3>Entities in this codebase</h3>
      <ul>
        <li><strong>Supplier</strong> — a company that provides ingredients.</li>
        <li>
          <strong>Ingredient</strong> — a product offered by a supplier, with a unit and price.
        </li>
        <li>
          <strong>Buyer</strong> — a buying company with a linked Django user account.
        </li>
        <li>
          <strong>Order</strong> — a set of order items (ingredient + quantity + snapshot price)
          placed by a buyer. Orders move through a state machine:
          <code>PENDING → CONFIRMED → PROCESSING → SHIPPED → DELIVERED</code> (or
          <code>CANCELLED</code> from PENDING / CONFIRMED).
        </li>
      </ul>
      <p class="session-prompt">
        During your session you will be given a <strong>specific use case</strong> to implement.
        Use this page, the API docs in the README, and the existing code as a starting point.
      </p>
    </section>

    <!-- Buyer profile -->
    <section class="profile-card" v-if="profile">
      <div>
        <p class="label">Company</p>
        <p class="value">{{ profile.company_name }}</p>
      </div>
      <div>
        <p class="label">Username</p>
        <p class="value">{{ profile.username }}</p>
      </div>
      <div>
        <p class="label">Total orders</p>
        <p class="value">{{ profile.total_orders }}</p>
      </div>
    </section>

    <!-- Quick stats -->
    <section class="stats">
      <h3>Orders by status</h3>
      <div class="stat-grid">
        <div v-for="s in ALL_STATUSES" :key="s" class="stat-item">
          <span :class="['badge', `badge--${s.toLowerCase()}`]">{{ s }}</span>
          <span class="stat-count">{{ statusCounts[s] ?? 0 }}</span>
        </div>
      </div>
    </section>

    <!-- Recent orders -->
    <section v-if="recentOrders.length">
      <h3>Recent orders</h3>
      <table class="orders-table">
        <thead>
          <tr>
            <th>#</th>
            <th>Status</th>
            <th>Items</th>
            <th>Total</th>
            <th>Created</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="order in recentOrders" :key="order.id">
            <td>{{ order.id }}</td>
            <td>
              <span :class="['badge', `badge--${order.status.toLowerCase()}`]">
                {{ order.status }}
              </span>
            </td>
            <td>{{ order.item_count }}</td>
            <td>£{{ order.total_amount }}</td>
            <td>{{ formatDate(order.created_at) }}</td>
          </tr>
        </tbody>
      </table>
    </section>
  </div>
</template>

<style scoped>
.dashboard {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.loading {
  padding: 2rem;
  color: #6b7280;
}

.challenge-overview {
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  border-radius: 8px;
  padding: 1.5rem;
}

.challenge-overview h2 {
  margin-top: 0;
  color: #1e40af;
}

.challenge-overview h3 {
  margin-top: 1rem;
}

.challenge-overview ul {
  padding-left: 1.25rem;
  line-height: 1.8;
}

.session-prompt {
  background: #fff;
  border-left: 4px solid #1a56db;
  padding: 0.75rem 1rem;
  border-radius: 0 6px 6px 0;
  margin-top: 1rem;
}

.profile-card {
  display: flex;
  gap: 2rem;
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 1.25rem 1.5rem;
}

.label {
  font-size: 0.75rem;
  color: #6b7280;
  margin: 0 0 0.25rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.value {
  font-weight: 600;
  font-size: 1.1rem;
  margin: 0;
}

.stat-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  margin-top: 0.75rem;
}

.stat-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
  border-radius: 6px;
  padding: 0.5rem 0.75rem;
}

.stat-count {
  font-weight: 700;
  font-size: 1.1rem;
}

.orders-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.orders-table th,
.orders-table td {
  padding: 0.6rem 0.75rem;
  text-align: left;
  border-bottom: 1px solid #e5e7eb;
}

.orders-table th {
  font-weight: 600;
  color: #374151;
  background: #f9fafb;
}

/* Status badges — shared via global in App.vue */
.badge {
  display: inline-block;
  padding: 0.2rem 0.6rem;
  border-radius: 9999px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.badge--pending { background: #fef9c3; color: #854d0e; }
.badge--confirmed { background: #dbeafe; color: #1e40af; }
.badge--processing { background: #ede9fe; color: #5b21b6; }
.badge--shipped { background: #ffedd5; color: #9a3412; }
.badge--delivered { background: #dcfce7; color: #166534; }
.badge--cancelled { background: #fee2e2; color: #991b1b; }
</style>
