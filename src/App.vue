<script setup>
import DollarTable from './components/DollarTable.vue'
</script>

<template>
  <div class="app">
    <header class="app-header">
      <h1>Dólares — Mini DolarHoy</h1>
      <p class="subtitle">Datos: DolarApi.com · Actualiza automáticamente cada 60s</p>
    </header>

    <main>
      <DollarTable />
    </main>

    <footer class="app-footer">
      <small>Fuente: DolarApi.com</small>
    </footer>
  </div>
</template>

<style scoped>
:root {
  --bg: #0f172a;
  --card: #ffffff;
  --muted: #6b7280;
}

body {
  font-family: Inter, system-ui, Arial;
  margin: 0;
  background: #f7fafc;
  color: #111827;
}

.app {
  max-width: 900px;
  margin: 24px auto;
  padding: 0 16px;
}

.app-header {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-bottom: 12px;
}

.subtitle {
  color: var(--muted);
  margin: 0;
  font-size: 0.9rem;
}

.app-footer {
  margin-top: 18px;
  color: var(--muted);
  text-align: center;
}
</style>
