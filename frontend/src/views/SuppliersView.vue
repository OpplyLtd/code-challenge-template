<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import type { Supplier } from "../types";
import { fetchSuppliers } from "../services/suppliers";

const router = useRouter();
const suppliers = ref<Supplier[]>([]);
const loading = ref(true);

onMounted(async () => {
  suppliers.value = await fetchSuppliers();
  loading.value = false;
});

function goToDetail(id: number) {
  router.push({ name: "supplier-detail", params: { id } });
}
</script>

<template>
  <div>
    <h2>Suppliers</h2>
    <div v-if="loading" class="loading">Loading…</div>
    <div v-else class="grid">
      <div
        v-for="supplier in suppliers"
        :key="supplier.id"
        class="card"
        @click="goToDetail(supplier.id)"
      >
        <h3>{{ supplier.name }}</h3>
        <p class="description">{{ supplier.description }}</p>
        <div class="meta">
          <span class="ingredient-count">{{ supplier.ingredient_count }} ingredients</span>
          <span class="view-link">View details →</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
h2 {
  margin-top: 0;
}

.loading {
  color: #6b7280;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.25rem;
}

.card {
  background: #fff;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 1.25rem;
  cursor: pointer;
  transition: box-shadow 0.15s, border-color 0.15s;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
  border-color: #1a56db;
}

.card h3 {
  margin: 0;
  font-size: 1rem;
  color: #111827;
}

.description {
  margin: 0;
  font-size: 0.85rem;
  color: #4b5563;
  line-height: 1.5;
  flex: 1;
}

.meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 0.5rem;
}

.ingredient-count {
  font-size: 0.8rem;
  color: #6b7280;
  background: #f3f4f6;
  padding: 0.2rem 0.6rem;
  border-radius: 9999px;
}

.view-link {
  font-size: 0.8rem;
  color: #1a56db;
  font-weight: 500;
}
</style>
