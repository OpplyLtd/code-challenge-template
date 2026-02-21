<script setup lang="ts">
import { ref, onMounted } from "vue";
import type { Order } from "../types";
import { fetchOrders } from "../services/orders";

const orders = ref<Order[]>([]);
const loading = ref(true);

onMounted(async () => {
  orders.value = await fetchOrders();
  loading.value = false;
});

function formatDate(iso: string): string {
  return new Date(iso).toLocaleDateString("en-GB", {
    day: "numeric",
    month: "short",
    year: "numeric",
  });
}
</script>

<template>
  <div>
    <h2>My Orders</h2>
    <div v-if="loading" class="loading">Loading…</div>
    <div v-else-if="orders.length === 0" class="empty">No orders yet.</div>
    <table v-else class="orders-table">
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
        <tr v-for="order in orders" :key="order.id">
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
  </div>
</template>

<style scoped>
h2 {
  margin-top: 0;
}

.loading,
.empty {
  color: #6b7280;
}

.orders-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
}

.orders-table th,
.orders-table td {
  padding: 0.65rem 0.75rem;
  text-align: left;
  border-bottom: 1px solid #e5e7eb;
}

.orders-table th {
  font-weight: 600;
  color: #374151;
  background: #f9fafb;
}

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
