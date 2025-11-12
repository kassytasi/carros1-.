<template>
  <q-page class="q-pa-md bg-grey-1">

    <!-- Alerta grande -->
    <q-banner v-if="alertaGrande" class="bg-yellow-2 text-yellow-10 q-mb-md">
      ⚠️ <strong>Compra grande detectada:</strong>
      Tu carrito supera los <b>$1000</b>. ¡Podrías calificar para envío gratis!
    </q-banner>

    <!-- Alerta carrito guardado -->
    <q-banner v-if="carritoGuardado" class="bg-green-2 text-green-10 q-mb-md">
      ✅ Carrito guardado automáticamente en localStorage
    </q-banner>

    <!-- Título -->
    <h4 class="text-primary text-bold q-mb-md">🛍️ Productos Disponibles</h4>

    <!-- Lista de productos -->
    <div class="row q-col-gutter-md">
      <div v-for="(p, i) in productos" :key="i" class="col-12 col-sm-6 col-md-4">
        <q-card
          class="producto-card cursor-pointer"
          flat
          bordered
          @mouseover="hoverIndex = i"
          @mouseleave="hoverIndex = null"
        >
          <q-card-section class="text-center">
            <div class="text-h6 q-mb-xs">
              <q-icon :name="p.icono" size="24px" class="q-mr-sm text-primary" />
              {{ p.nombre }}
            </div>
            <div class="text-h6 text-positive">${{ p.precio }}</div>
          </q-card-section>

          <q-separator />

          <q-card-actions align="around">
            <q-btn
              round
              color="negative"
              icon="remove"
              @click="disminuir(i)"
              class="scale-btn"
            />
            <div class="text-bold text-primary text-h6">{{ p.cantidad }}</div>
            <q-btn
              round
              color="positive"
              icon="add"
              @click="aumentar(i)"
              class="scale-btn"
            />
          </q-card-actions>
        </q-card>
      </div>
    </div>

    <!-- Totales -->
    <div class="q-mt-lg text-center">
      <q-separator class="q-my-md" />
      <div class="text-h6">Subtotal: ${{ subtotal }}</div>
      <div class="text-h6">Impuesto (16%): ${{ impuesto }}</div>
      <div class="text-h5 text-bold text-positive">Total Final: ${{ totalFinal }}</div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

// Productos iniciales con íconos y emojis 🎨
const productos = ref([
  { nombre: '💻 Laptop Dell', precio: 899, cantidad: 2, icono: 'laptop' },
  { nombre: '🖱️ Mouse Logitech', precio: 25, cantidad: 2, icono: 'mouse' },
  { nombre: '⌨️ Teclado Mecánico', precio: 89, cantidad: 1, icono: 'keyboard' }
])

const hoverIndex = ref(null)
const carritoGuardado = ref(true)
const alertaGrande = ref(false)

// Funciones para cambiar cantidades
const aumentar = (i) => productos.value[i].cantidad++
const disminuir = (i) => {
  if (productos.value[i].cantidad > 0) productos.value[i].cantidad--
}

// Computadas
const subtotal = computed(() =>
  productos.value.reduce((t, p) => t + p.precio * p.cantidad, 0)
)
const impuesto = computed(() => +(subtotal.value * 0.16).toFixed(2))
const totalFinal = computed(() => +(subtotal.value + impuesto.value).toFixed(2))

// Watch para detectar compra grande
watch(totalFinal, (nuevo) => {
  alertaGrande.value = nuevo > 1000
})
</script>

<style scoped>
.producto-card {
  transition: all 0.3s ease;
  border: 2px solid transparent;
  background-color: white;
}

.producto-card:hover {
  transform: translateY(-4px);
  border-color: #1976d2;
  box-shadow: 0 8px 16px rgba(25, 118, 210, 0.15);
  background-color: #f9fbff;
}

/* Animación suave en botones */
.scale-btn {
  transition: transform 0.2s ease;
}
.scale-btn:hover {
  transform: scale(1.2);
}
</style>


