<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import type { Supplier, Ingredient } from "../types";
import { fetchSupplier, fetchSupplierIngredients } from "../services/suppliers";

const route = useRoute();
const router = useRouter();

const supplier = ref<Supplier | null>(null);
const ingredients = ref<Ingredient[]>([]);
const loading = ref(true);

onMounted(async () => {
  const id = Number(route.params.id);
  const [s, ings] = await Promise.all([
    fetchSupplier(id),
    fetchSupplierIngredients(id),
  ]);
  supplier.value = s;
  ingredients.value = ings;
  loading.value = false;
});
</script>

<template>
  <div>
    <button class="back-btn" @click="router.back()">← Back to Suppliers</button>

    <div v-if="loading" class="loading">Loading…</div>
    <div v-else-if="supplier">
      <h2>{{ supplier.name }}</h2>
      <p class="description">{{ supplier.description }}</p>

      <h3>Ingredients ({{ ingredients.length }})</h3>
      <table class="ingredients-table">
        <thead>
          <tr>
            <th>Name</th>
            <th>Description</th>
            <th>Unit</th>
            <th>Price / unit</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="ing in ingredients" :key="ing.id">
            <td>{{ ing.name }}</td>
            <td class="desc-cell">{{ ing.description }}</td>
            <td>{{ ing.unit }}</td>
            <td>£{{ ing.price_per_unit }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.back-btn {
  background: none;
  border: none;
  color: #1a56db;
  font-size: 0.875rem;
  cursor: pointer;
  padding: 0;
  margin-bottom: 1.25rem;
}

.back-btn:hover {
  text-decoration: underline;
}

.loading {
  color: #6b7280;
}

h2 {
  margin-top: 0;
}

.description {
  color: #4b5563;
  max-width: 700px;
  line-height: 1.6;
}

.ingredients-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.9rem;
  margin-top: 0.75rem;
}

.ingredients-table th,
.ingredients-table td {
  padding: 0.65rem 0.75rem;
  text-align: left;
  border-bottom: 1px solid #e5e7eb;
}

.ingredients-table th {
  font-weight: 600;
  color: #374151;
  background: #f9fafb;
}

.desc-cell {
  color: #6b7280;
  font-size: 0.85rem;
  max-width: 300px;
}
</style>
