<template>
    <div class="card">
        <div class="card-top">
            <h2></h2>
            <div class="controls">
                <button @click="fetchData" :disabled="loading">Actualizar</button>
                <span v-if="lastUpdate" class="last">Última: {{ lastUpdate }}</span>
            </div>
        </div>

        <div v-if="error" class="error">Error: {{ error }}</div>

        <table v-else class="dolares-table" aria-live="polite">
            <thead>
                <tr>
                    <th>Tipo</th>
                    <th>Compra</th>
                    <th>Venta</th>
                    <th>Brecha</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="row in rows" :key="row.casa">
                    <td class="tipo">
                        <span class="dot" :style="{ backgroundColor: row.color }"></span>
                        <strong>{{ row.nombre }}</strong>
                    </td>
                    <td class="num">{{ formatMoney(row.compra) }}</td>
                    <td class="num">{{ formatMoney(row.venta) }}</td>
                    <td class="num">{{ row.brecha === null ? '-' : formatBrecha(row.brecha) }}</td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const rows = ref([])
const loading = ref(false)
const error = ref(null)
const lastUpdate = ref(null)
const refreshIntervalsMs = 60_000 // refresco cada 60 segundos

const casasMap = {
    oficial: 'Oficial',
    blue: 'Blue',
    bolsa: 'MEP',
    contadoconliqui: 'CCL',
    mayorista: 'Mayorista',
    tarjeta: 'Tarjeta',
    cripto: 'Cripto'
}

const coloresMap = {
    oficial: '#6b7280',
    blue: '#0ea5e9',
    bolsa: '#7c3aed',
    contadoconliqui: '#10b981',
    mayorista: '#f472b6',
    tarjeta: '#f59e0b',
    cripto: '#f59e0b'
}

// ENDPOINT (DolarApi)
const API_URL = 'https://dolarapi.com/v1/dolares'

const formatMoney = (v) => {
    if (v === null || v === undefined || Number.isNaN(Number(v))) return '-'
    return `$ ${Number(v).toLocaleString('es-AR', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`
}

const formatBrecha = (b) => `${b > 0 ? '+' : ''}${b.toFixed(2)}`

async function fetchData() {
    loading.value = true
    error.value = null
    try {
        const res = await fetch(API_URL)
        if (!res.ok) throw new Error(`HTTP ${res.status}`)
        const data = await res.json()

        // Buscar el oficial para calcular brechas
        const oficial = data.find(d => d.casa === 'oficial')
        const oficialVenta = oficial ? Number(oficial.venta) : null

        // formatear rows
        const mapped = data.map(d => {
            const venta = d.venta == null ? null : Number(d.venta)
            const compra = d.compra == null ? null : Number(d.compra)
            const brecha = oficialVenta == null || d.casa === 'oficial' ? 0 : ((venta - oficialVenta) / oficialVenta) * 100
            return {
                casa: d.casa,
                nombre: casasMap[d.casa] ?? (d.nombre ?? d.casa),
                compra,
                venta,
                fecha: d.fechaActualizacion ?? d.fecha ?? null,
                brecha: d.casa === 'oficial' ? 0 : Number(brecha),
                color: coloresMap[d.casa] ?? '#94a3b8'
            }
        })

        // orden preferido (mostramos primero los más relevantes)
        const order = ['oficial', 'blue', 'bolsa', 'contadoconliqui', 'tarjeta', 'mayorista', 'cripto']
        mapped.sort((a, b) => {
            const ia = order.indexOf(a.casa)
            const ib = order.indexOf(b.casa)
            if (ia === -1 && ib === -1) return a.nombre.localeCompare(b.nombre)
            if (ia === -1) return 1
            if (ib === -1) return -1
            return ia - ib
        })

        rows.value = mapped
        lastUpdate.value = mapped.find(r => r.fecha)?.fecha ? new Date(mapped.find(r => r.fecha).fecha).toLocaleString('es-AR') : new Date().toLocaleString('es-AR')

    } catch (e) {
        console.error(e)
        error.value = e.message || 'Error desconocido'
    } finally {
        loading.value = false
    }
}
onMounted(() => {
    fetchData()
    setInterval(fetchData, refreshIntervalsMs)
})
</script>

<style scoped>
.card {
    background: white;
    padding: 14px;
    border-radius: 10px;
    box-shadow: 0 6px 18px rgba(15, 23, 42, 0.06);
}

.card-top {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
    gap: 10px;
}

.controls {
    display: flex;
    gap: 10px;
    align-items: center;
}

.last {
    color: #6b7280;
    font-size: 0.9rem;
}

.dolares-table {
    width: 100%;
    border-collapse: collapse;
}

.dolares-table thead th {
    text-align: left;
    padding: 10px;
    color: #374151;
    font-weight: 600;
    border-bottom: 1px solid #e6edf3;
}

.dolares-table tbody td {
    padding: 10px;
    border-bottom: 1px dashed #eef2f7;
    vertical-align: middle;
}

.tipo {
    display: flex;
    align-items: center;
    gap: 10px;
}

.dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    display: inline-block;
}

.num {
    font-family: 'Roboto Mono', monospace;
}

.tag {
    font-size: 0.75rem;
    color: #9ca3af;
    margin-left: 6px;
    display: inline-block;
}

.error {
    color: #b91c1c;
    margin: 8px 0;
}

button {
    padding: 6px 10px;
    border-radius: 8px;
    border: 1px solid #e6edf3;
    background: #fff;
    cursor: pointer;
}

button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
}

@media (max-width:640px) {
    .dolares-table thead {
        display: none;
    }

    .dolares-table tbody td {
        display: block;
        width: 100%;
    }

    .dolares-table tbody tr {
        margin-bottom: 10px;
        border: 1px solid #f1f5f9;
        border-radius: 8px;
        padding: 8px;
    }

    .tag {
        display: block;
        margin-top: 6px;
    }
}
</style>