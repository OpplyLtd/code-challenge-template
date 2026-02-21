<script setup lang="ts">
import { useRouter } from "vue-router";
import { useAuth } from "./composables/useAuth";

const router = useRouter();
const { isAuthenticated, clearToken } = useAuth();

function logout() {
  clearToken();
  router.push({ name: "login" });
}
</script>

<template>
  <div id="layout">
    <nav v-if="isAuthenticated" class="navbar">
      <span class="brand">Opply</span>
      <div class="nav-links">
        <router-link :to="{ name: 'dashboard' }">Dashboard</router-link>
        <router-link :to="{ name: 'suppliers' }">Suppliers</router-link>
        <router-link :to="{ name: 'orders' }">Orders</router-link>
      </div>
      <button class="logout-btn" @click="logout">Logout</button>
    </nav>
    <main :class="{ 'with-nav': isAuthenticated }">
      <router-view />
    </main>
  </div>
</template>

<style>
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Oxygen, sans-serif;
  background: #f9fafb;
  color: #111827;
}
</style>

<style scoped>
#layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.navbar {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  padding: 0 1.5rem;
  height: 56px;
  background: #fff;
  border-bottom: 1px solid #e5e7eb;
  position: sticky;
  top: 0;
  z-index: 100;
}

.brand {
  font-weight: 700;
  font-size: 1.1rem;
  color: #1a56db;
  margin-right: auto;
}

.nav-links {
  display: flex;
  gap: 1rem;
}

.nav-links a {
  text-decoration: none;
  font-size: 0.9rem;
  color: #4b5563;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  transition: color 0.15s, background 0.15s;
}

.nav-links a:hover {
  color: #1a56db;
  background: #eff6ff;
}

.nav-links a.router-link-active {
  color: #1a56db;
  font-weight: 600;
}

.logout-btn {
  background: none;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  padding: 0.35rem 0.8rem;
  font-size: 0.85rem;
  cursor: pointer;
  color: #374151;
  transition: border-color 0.15s, color 0.15s;
}

.logout-btn:hover {
  border-color: #ef4444;
  color: #ef4444;
}

main {
  flex: 1;
  padding: 2rem 1.5rem;
  max-width: 1100px;
  width: 100%;
  margin: 0 auto;
}
</style>
