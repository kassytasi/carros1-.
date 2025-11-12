<template>
  <q-layout view="hHh lpR fFf">
    <!-- 🔹 Header -->
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title class="text-h5">
          🛒 FuturoShop
        </q-toolbar-title>

        <!-- ❤️ Corazón con animación -->
        <div class="heart-notify q-ml-md">
          <q-icon name="favorite" size="xl" class="heart-anim" />
          <span v-if="totalItems > 0">{{ totalItems }}</span>
        </div>
      </q-toolbar>
    </q-header>

    <!-- 🔹 Contenido -->
    <q-page-container>
      <q-page class="q-pa-xl bg-grey-2">

        <div class="text-center q-mb-lg">
          <h1 class="text-h4 text-primary">Carrito de Compras</h1>
          <p class="text-grey-7">Agrega productos y observa cómo los totales se actualizan automáticamente</p>
        </div>

        <!-- 🔸 Alertas -->
        <div class="q-mb-md">
          <q-alert v-if="alertaGrande" color="warning" icon="warning" dense>
            ⚠️ Compra grande: tu total supera los $1000. ¡Podrías obtener envío gratis!
          </q-alert>
          <q-alert v-if="guardado" color="positive" icon="save" dense>
            💾 Carrito guardado en localStorage
          </q-alert>
        </div>

        <!-- 🔸 Cuerpo -->
        <div class="row q-col-gutter-xl">
          <!-- 🧱 Productos -->
          <div class="col-12 col-md-7">
            <q-card flat bordered class="q-pa-md">
              <div class="text-h6 text-primary q-mb-md">Productos</div>

              <div v-for="(producto, index) in productos" :key="index" class="q-mb-md">
                <!-- 🌟 Efecto hover -->
                <q-card
                  bordered
                  class="q-pa-md row items-center justify-between card-hover"
                  :class="{ pulse: producto.animar }"
                  @mouseover="producto.hover = true"
                  @mouseleave="producto.hover = false"
                >

                  <!-- Emoji e info -->
                  <div class="row items-center q-gutter-md">
                    <div class="emoji text-h4">{{ producto.emoji }}</div>
                    <div>
                      <div class="text-subtitle1 text-dark">{{ producto.nombre }}</div>
                      <div class="text-bold text-positive">${{ producto.precio }}</div>
                    </div>
                  </div>

                  <!-- Botones -->
                  <transition name="fade">
                    <div>
                      <q-btn
                        v-if="!producto.agregado"
                        color="primary"
                        label="Agregar al carrito"
                        @click="agregarAlCarrito(producto)"
                        glossy
                      />
                      <div v-else class="row items-center q-gutter-sm">
                        <q-btn round dense color="negative" icon="remove" @click="disminuir(producto)" />
                        <div class="text-body1 text-bold">{{ producto.cantidad }}</div>
                        <q-btn round dense color="positive" icon="add" @click="aumentar(producto)" />
                      </div>
                    </div>
                  </transition>
                </q-card>
              </div>
            </q-card>
          </div>

          <!-- 🧾 Resumen -->
          <div class="col-12 col-md-5">
            <q-card bordered flat class="q-pa-lg resumen">
              <div class="text-h6 text-primary q-mb-sm">Resumen del Carrito</div>
              <q-separator spaced />
              <div><strong>Productos:</strong> {{ totalItems }} ítems</div>
              <div><strong>Subtotal:</strong> ${{ subtotal.toFixed(2) }}</div>
              <div><strong>Impuesto (16%):</strong> ${{ impuesto.toFixed(2) }}</div>
              <q-separator spaced />
              <div class="text-h6 text-positive text-bold q-mt-sm">
                Total a pagar: ${{ totalFinal.toFixed(2) }}
              </div>
              <div class="q-mt-md text-center">
                <q-btn color="primary" icon="credit_card" label="Finalizar compra" size="lg" glossy :disable="totalItems === 0" />
                <q-btn color="negative" icon="delete" label="Vaciar carrito" flat class="q-mt-sm" @click="vaciarCarrito" :disable="totalItems === 0"/>
              </div>
            </q-card>
          </div>
        </div>

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const productos = ref([
  { nombre: 'Smartphone Galaxy S25', precio: 1200, cantidad: 0, agregado: false, emoji: '📱', hover: false, animar: false },
  { nombre: 'Audífonos Bose', precio: 350, cantidad: 0, agregado: false, emoji: '🎧', hover: false, animar: false },
  { nombre: 'Tablet iPad Pro', precio: 999, cantidad: 0, agregado: false, emoji: '💻', hover: false, animar: false },
  { nombre: 'Smartwatch Apple', precio: 450, cantidad: 0, agregado: false, emoji: '⌚', hover: false, animar: false },
  { nombre: 'Cámara GoPro Hero12', precio: 499, cantidad: 0, agregado: false, emoji: '📸', hover: false, animar: false }
])

const guardado = ref(false)
const alertaGrande = ref(false)

const totalItems = computed(() => productos.value.reduce((t, p) => t + p.cantidad, 0))
const subtotal = computed(() => productos.value.reduce((t, p) => t + p.precio * p.cantidad, 0))
const impuesto = computed(() => subtotal.value * 0.16)
const totalFinal = computed(() => subtotal.value + impuesto.value)

function agregarAlCarrito(producto) {
  producto.cantidad = 1
  producto.agregado = true
  animar(producto)
  guardarCarrito()
}

function aumentar(producto) {
  producto.cantidad++
  animar(producto)
  guardarCarrito()
}

function disminuir(producto) {
  if (producto.cantidad > 0) {
    producto.cantidad--
    if (producto.cantidad === 0) producto.agregado = false
    guardarCarrito()
  }
}

function vaciarCarrito() {
  productos.value.forEach(p => {
    p.cantidad = 0
    p.agregado = false
  })
  guardarCarrito()
}

function guardarCarrito() {
  localStorage.setItem('carrito', JSON.stringify(productos.value))
  guardado.value = true
  setTimeout(() => (guardado.value = false), 1500)
}

function animar(producto) {
  producto.animar = true
  setTimeout(() => (producto.animar = false), 300)
}

onMounted(() => {
  const guardadoCarrito = localStorage.getItem('carrito')
  if (guardadoCarrito) {
    const parsed = JSON.parse(guardadoCarrito)
    productos.value = productos.value.map(p => {
      const saved = parsed.find(sp => sp.nombre === p.nombre)
      return saved ? { ...p, cantidad: saved.cantidad, agregado: saved.agregado } : p
    })
  }
})

watch(totalFinal, nuevoTotal => {
  alertaGrande.value = nuevoTotal > 1000
})
</script>

<style scoped lang="sass">
h1, h2, .text-h6
  font-family: "Inter", sans-serif

/* ✨ Tarjetas de productos con efectos hover */
.card-hover
  transition: all 0.3s ease
  cursor: pointer
  &:hover
    transform: translateY(-4px)
    box-shadow: 0 8px 20px rgba(0,0,0,0.15)
    border-color: #9C27B0

/* 💥 Efecto de pulso cuando se agrega */
.pulse
  animation: pulse 0.3s ease

@keyframes pulse
  0%
    transform: scale(1)
  50%
    transform: scale(1.05)
  100%
    transform: scale(1)

/* ❤️ Corazón con animación suave */
.heart-anim
  transition: transform 0.2s ease
  &:hover
    transform: scale(1.2)
    color: #E040FB

/* Fade para botones */
.fade-enter-active, .fade-leave-active
  transition: opacity 0.3s ease
.fade-enter-from, .fade-leave-to
  opacity: 0

.emoji
  font-size: 2rem
</style>

