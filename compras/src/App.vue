<template>
  <q-layout view="hHh lpR fFf">

    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-toolbar-title class="text-h5 text-weight-bold">
          🛒 shopping
        </q-toolbar-title>

    
        <div class="heart-notify q-ml-md">
          <q-icon 
            :name="totalFavoritos > 0 ? 'favorite' : 'favorite_border'" 
            size="xl" 
            class="heart-anim" 
            :class="{ 'text-pink': totalFavoritos > 0 }"
          />
          <span v-if="totalItems > 0" class="badge">{{ totalItems }}</span>
          <span v-if="totalFavoritos > 0" class="badge-favoritos">{{ totalFavoritos }}</span>
        </div>
      </q-toolbar>
    </q-header>

    <!-- 🔹 Contenido -->
    <q-page-container>
      <q-page class="q-pa-md q-pa-xl--lg bg-grey-2">

        <div class="text-center q-mb-lg">
          <h1 class="text-h4 text-h3-lg text-primary text-weight-bold">Carrito de Compras</h1>
          <p class="text-grey-7 text-body1 text-subtitle1-lg">Agrega productos y observa cual te interesa mas</p>
        </div>

        <!-- 🔸 Alertas APILADAS -->
        <div class="q-mb-md alertas-container">
          <q-banner v-if="alertaGrande" class="bg-warning text-white q-mb-sm">
            <template v-slot:avatar>
              <q-icon name="warning" />
            </template>
            ⚠️ Compra grande: tu total supera los $1000. ¡Podrías obtener envío gratis!
          </q-banner>

          <q-banner v-if="guardado" class="bg-positive text-white q-mb-sm">
            <template v-slot:avatar>
              <q-icon name="save" />
            </template>
            💾 Carrito guardado en localStorage
          </q-banner>

          <q-banner v-if="compraFinalizada" class="bg-info text-white">
            <template v-slot:avatar>
              <q-icon name="check_circle" />
            </template>
            ✅ ¡Compra finalizada con éxito! Gracias por tu compra
          </q-banner>

          <q-banner v-if="favoritoAgregado" class="bg-pink text-white">
            <template v-slot:avatar>
              <q-icon name="favorite" />
            </template>
            ❤️ Producto agregado a favoritos
          </q-banner>
        </div>

        <div class="row q-col-gutter-lg q-col-gutter-xl-lg">
          <!-- 🧱 Productos -->
          <div class="col-12 col-md-7">
            <q-card flat bordered class="q-pa-md card-productos">
              <q-card-section class="q-pb-none">
                <div class="text-h6 text-primary text-weight-bold">Productos Disponibles</div>
                <p class="text-grey-7 q-mt-sm">Selecciona los productos que deseas agregar a tu carrito</p>
              </q-card-section>

              <q-card-section>
                <div v-for="(producto, index) in productos" :key="index" class="q-mb-md">
                  <q-card
                    bordered
                    class="q-pa-sm q-pa-md-md row items-center justify-between card-hover"
                    :class="{ pulse: producto.animar, 'card-producto': true }"
                    @mouseover="producto.hover = true"
                    @mouseleave="producto.hover = false"
                  >
                    <!-- Imagen e info -->
                    <div class="row items-center q-gutter-sm q-gutter-md-md">
                      <div class="producto-imagen" @click="abrirModalImagen(producto)">
                        <img 
                          :src="getImageUrl(producto.imagen)" 
                          :alt="producto.nombre"
                          class="imagen-producto clickable"
                          @error="handleImageError"
                        />
                        <div class="overlay">
                          <q-icon name="zoom_in" size="md" class="zoom-icon" />
                        </div>
                      </div>
                      <div class="q-ml-sm">
                        <div class="text-subtitle2 text-subtitle1-md text-dark text-weight-medium">{{ producto.nombre }}</div>
                        <div class="text-caption text-body2-md text-grey-7 q-mb-xs">{{ producto.descripcion }}</div>
                        <div class="text-bold text-positive text-h6 text-h5-md">${{ producto.precio }}</div>
                        
                     
                        <q-btn
                          round
                          dense
                          :icon="producto.favorito ? 'favorite' : 'favorite_border'"
                          :color="producto.favorito ? 'pink' : 'grey-6'"
                          size="sm"
                          class="q-mt-xs"
                          @click="toggleFavorito(producto)"
                        >
                          <q-tooltip>
                            {{ producto.favorito ? 'Quitar de favoritos' : 'Agregar a favoritos' }}
                          </q-tooltip>
                        </q-btn>
                      </div>
                    </div>

                    <!-- Botones -->
                    <div class="q-mt-xs q-mt-none-md">
                      <q-btn
                        v-if="!producto.agregado"
                        color="purple-4"
                        :label="screen.gt.xs ? 'Agregar al carrito' : 'Agregar'"
                        @click="agregarAlCarrito(producto)"
                        glossy
                        class="btn-agregar-morado"
                        :size="screen.lt.sm ? 'sm' : 'md'"
                      />
                      <div v-else class="row items-center q-gutter-xs q-gutter-sm-md">
                        <q-btn 
                          round 
                          dense 
                          color="negative" 
                          icon="remove" 
                          @click="disminuir(producto)" 
                          class="btn-disminuir"
                          :size="screen.lt.sm ? 'sm' : 'md'"
                        />
                        <div class="text-body1 text-bold cantidad">{{ producto.cantidad }}</div>
                        <q-btn 
                          round 
                          dense 
                          color="positive" 
                          icon="add" 
                          @click="aumentar(producto)" 
                          class="btn-aumentar"
                          :size="screen.lt.sm ? 'sm' : 'md'"
                        />
                      </div>
                    </div>
                  </q-card>
                </div>
              </q-card-section>
            </q-card>
          </div>

       
          <div class="col-12 col-md-5">
            <q-card bordered flat class="q-pa-lg resumen">
              <!-- ❤️ Corazón en el resumen -->
              <div class="row items-center justify-center q-mb-md">
                <div class="heart-notify-resumen">
                  <q-icon 
                    :name="totalFavoritos > 0 ? 'favorite' : 'favorite_border'" 
                    size="xl" 
                    class="heart-anim-resumen" 
                    :class="{ 'text-pink': totalFavoritos > 0 }"
                  />
                  <span v-if="totalItems > 0" class="badge-resumen">{{ totalItems }}</span>
                  <span v-if="totalFavoritos > 0" class="badge-favoritos-resumen">{{ totalFavoritos }}</span>
                </div>
              </div>
              
              <div class="text-h6 text-primary text-weight-bold text-center">Resumen del Carrito</div>
              <p class="text-grey-7 text-caption text-center q-mb-md">Revisa los detalles de tu compra</p>
              
              <q-separator spaced />
              
            
              <div class="resumen-detalles">
                <div class="row items-center justify-between q-mb-sm">
                  <span class="text-blue-grey-7 text-body2 text-body1-md text-weight-medium">TOTAL DE PRODUCTOS:</span>
                  <span class="text-bold text-indigo-10 text-body1 text-h6-md">{{ totalItems }} ítems</span>
                </div>
                
                <div class="row items-center justify-between q-mb-sm">
                  <span class="text-blue-grey-7 text-body2 text-body1-md text-weight-medium">SUBTOTAL:</span>
                  <span class="text-bold text-deep-orange-8 text-body1 text-h6-md">${{ subtotal.toFixed(2) }}</span>
                </div>
                
            
                <div class="row items-center justify-between q-mb-sm">
                  <span class="text-blue-grey-7 text-body2 text-body1-md text-weight-medium">ENVÍO:</span>
                  <span v-if="envioGratis" class="text-bold text-positive text-body1 text-h6-md">
                    GRATIS 🎉
                  </span>
                  <span v-else class="text-bold text-deep-orange-8 text-body1 text-h6-md">
                    ${{ costoEnvio.toFixed(2) }}
                  </span>
                </div>

              
                <div v-if="!envioGratis && subtotal > 0" class="row items-center justify-between q-mb-sm">
                  <span class="text-caption text-blue-grey-6 text-weight-medium">
                    ¡Faltan ${{ (1000 - subtotal).toFixed(2) }} para envío gratis!
                  </span>
                </div>
                
                <div class="row items-center justify-between q-mb-sm">
                  <span class="text-blue-grey-7 text-body2 text-body1-md text-weight-medium">IMPUESTO (16%):</span>
                  <span class="text-bold text-amber-8 text-body1 text-h6-md">${{ impuesto.toFixed(2) }}</span>
                </div>

                <div class="row items-center justify-between q-mb-sm">
                  <span class="text-blue-grey-7 text-body2 text-body1-md text-weight-medium">PRODUCTOS FAVORITOS:</span>
                  <span class="text-bold text-pink text-body1 text-h6-md">{{ totalFavoritos }} ítems</span>
                </div>
                
                <q-separator spaced class="q-my-md" />
                
                <div class="row items-center justify-between">
                  <span class="text-h6 text-primary text-weight-bold">TOTAL A PAGAR:</span>
                  <span class="text-h5 text-positive text-weight-bold">${{ totalFinal.toFixed(2) }}</span>
                </div>
              </div>

              <!-- 🎯 Botones -->
              <div class="q-mt-xl">
                <q-btn 
                  icon="credit_card" 
                  :label="screen.lt.sm ? 'Finalizar' : 'Finalizar compra'"
                  :size="screen.lt.sm ? 'md' : 'lg'"
                  glossy 
                  :disable="totalItems === 0" 
                  class="btn-finalizar-cool full-width q-mb-sm" 
                  @click="finalizarCompra"
                />
                <q-btn 
                  icon="delete" 
                  :label="screen.lt.sm ? 'Vaciar' : 'Vaciar carrito'"
                  :size="screen.lt.sm ? 'md' : 'lg'"
                  flat 
                  class="btn-vaciar-cool full-width q-mb-sm" 
                  @click="vaciarCarrito" 
                  :disable="totalItems === 0"
                />
                <q-btn 
                  icon="favorite" 
                  :label="screen.lt.sm ? 'Favoritos' : 'Ver favoritos'"
                  :size="screen.lt.sm ? 'md' : 'lg'"
                  outline
                  color="pink"
                  class="full-width" 
                  @click="verFavoritos"
                  :disable="totalFavoritos === 0"
                />
              </div>
            </q-card>
          </div>
        </div>

      </q-page>
    </q-page-container>

   
    <q-dialog v-model="modalImagen" maximized>
      <q-card class="modal-imagen-card">
        
        <q-bar class="bg-primary text-white modal-header">
          <div class="text-h6">{{ productoSeleccionado?.nombre }}</div>
          <q-space />
          <q-btn dense flat icon="close" v-close-popup>
            <q-tooltip>Cerrar</q-tooltip>
          </q-btn>
        </q-bar>

        <q-card-section class="q-pt-none modal-content">
         
          <div class="imagen-ampliada-container">
            <img 
              :src="getImageUrl(productoSeleccionado?.imagen)" 
              :alt="productoSeleccionado?.nombre"
              class="imagen-ampliada"
              @error="handleImageError"
            />
          </div>

         
          <div class="thumbnails-container q-mt-md">
            <div 
              v-for="(producto, index) in productos" 
              :key="index"
              class="thumbnail-item"
              :class="{ active: productoSeleccionado?.nombre === producto.nombre }"
              @click="seleccionarProducto(producto)"
            >
              <img 
                :src="getImageUrl(producto.imagen)" 
                :alt="producto.nombre"
                class="thumbnail-img"
              />
            </div>
          </div>

          <!-- Información del producto -->
          <div class="producto-info q-mt-lg">
            <div class="text-h5 text-weight-bold text-primary">{{ productoSeleccionado?.nombre }}</div>
            <div class="text-body1 text-grey-7 q-mb-sm">{{ productoSeleccionado?.descripcion }}</div>
            
          
            <div class="precio-section">
              <div class="text-h4 text-weight-bold text-positive">${{ productoSeleccionado?.precio }}</div>
              <div class="text-caption text-grey-6 text-line-through" v-if="productoSeleccionado?.precioOriginal">
                ${{ productoSeleccionado?.precioOriginal }}
              </div>
              <q-badge v-if="productoSeleccionado?.descuento" color="orange" class="q-ml-sm">
                {{ productoSeleccionado.descuento }}% OFF
              </q-badge>
            </div>

        
            <div class="detalles-adicionales q-mt-md">
              <div class="row items-center q-mb-xs">
                <q-icon name="local_shipping" :color="envioGratis ? 'positive' : 'grey-6'" class="q-mr-sm" />
                <span v-if="envioGratis" class="text-body2 text-weight-medium text-positive">
                  🎉 ¡Llega GRATIS mañana!
                </span>
                <span v-else class="text-body2 text-weight-medium">
                  📦 Envío: ${{ costoEnvio.toFixed(2) }} (Compra ${{ (1000 - subtotal).toFixed(2) }} más para GRATIS)
                </span>
              </div>
              <div class="row items-center q-mb-xs">
                <q-icon name="assignment_return" color="info" class="q-mr-sm" />
                <span class="text-body2 text-weight-medium">Devolución gratis - 30 días</span>
              </div>
              <div class="row items-center">
                <q-icon name="inventory_2" color="deep-purple" class="q-mr-sm" />
                <span class="text-body2 text-weight-medium">Stock disponible: <strong>+50 unidades</strong></span>
              </div>
            </div>

          
            <div class="botones-modal q-mt-lg">
              <q-btn 
                icon="add_shopping_cart" 
                label="Agregar al carrito"
                color="purple-4"
                glossy
                class="full-width q-mb-sm"
                @click="agregarDesdeModal"
                size="lg"
              />
              <q-btn 
                :icon="productoSeleccionado?.favorito ? 'favorite' : 'favorite_border'" 
                :label="productoSeleccionado?.favorito ? 'Quitar de favoritos' : 'Agregar a favoritos'"
                :color="productoSeleccionado?.favorito ? 'pink' : 'grey-7'"
                outline
                class="full-width"
                size="lg"
                @click="toggleFavoritoModal"
              />
            </div>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()
const screen = $q.screen

// 🔹 Variables reactivas para el modal
const modalImagen = ref(false)
const productoSeleccionado = ref(null)
const favoritoAgregado = ref(false)


const getImageUrl = (imageName) => {
  return new URL(`./assets/${imageName}`, import.meta.url).href
}

const handleImageError = (event) => {
  console.log('Error cargando imagen:', event.target.src)
  event.target.src = 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iODAiIGhlaWdodD0iODAiIHZpZXdCb3g9IjAgMCA4MCA4MCIgZmlsbD0ibm9uZSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj4KPHJlY3Qgd2lkdGg9IjgwIiBoZWlnaHQ9IjgwIiByeD0iOCIgZmlsbD0iI0Y1RjVGNSIvPgo8cGF0aCBkPSJNNDAgNDBDNDEuNjU2OSA0MCA0MyAzOC42NTY5IDQzIDM3QzQzIDM1LjM0MzEgNDEuNjU2OSAzNCA0MCAzNEMzOC4zNDMxIDM0IDM3IDM1LjM0MzEgMzcgMzdDMzcgMzguNjU2OSAzOC4zNDMxIDQwIDQwIDQwWiIgZmlsbD0iI0JEQkRCRCIvPgo8cGF0aCBkPSJNMzAgMjZINTBDNTEuMTA0NiAyNiA1MiAyNi44OTU0IDUyIDI4VjUyQzUyIDUzLjEwNDYgNTEuMTA0NiA1NCA1MCA1NEgzMEMyOC44OTU0IDU0IDI4IDUzLjEwNDYgMjggNTJWMjhDMjggMjYuODk1NCAyOC44OTU0IDI2IDMwIDI2WiIgZmlsbD0iI0JEQkRCRCIvPgo8L3N2Zz4K'
}

const productos = ref([
  { 
    nombre: 'Smartphone Galaxy S25', 
    precio: 1200, 
    precioOriginal: 1799,
    descuento: 33,
    cantidad: 0, 
    agregado: false, 
    favorito: false,
    imagen: 'celular.png', 
    hover: false, 
    animar: false,
    descripcion: 'Último modelo con cámara de 200MP' 
  },
  { 
    nombre: 'Audífonos Bose', 
    precio: 350, 
    precioOriginal: 700,
    descuento: 50,
    cantidad: 0, 
    agregado: false, 
    favorito: false,
    imagen: 'bose.png', 
    hover: false, 
    animar: false,
    descripcion: 'Cancelación de ruido premium' 
  },
  { 
    nombre: 'Tablet iPad Pro', 
    precio: 999, 
    precioOriginal: 1299,
    descuento: 23,
    cantidad: 0, 
    agregado: false, 
    favorito: false,
    imagen: 'table.png', 
    hover: false, 
    animar: false,
    descripcion: 'Pantalla Liquid Retina de 12.9"' 
  },
  { 
    nombre: 'Smartwatch Apple', 
    precio: 450, 
    precioOriginal: 599,
    descuento: 25,
    cantidad: 0, 
    agregado: false, 
    favorito: false,
    imagen: 'apple.png', 
    hover: false, 
    animar: false,
    descripcion: 'Monitoreo de salud avanzado' 
  },
  { 
    nombre: 'Cámara GoPro Hero12', 
    precio: 499, 
    precioOriginal: 649,
    descuento: 23,
    cantidad: 0, 
    agregado: false, 
    favorito: false,
    imagen: 'camara.png', 
    hover: false, 
    animar: false,
    descripcion: '4K 120fps, resistente al agua' 
  }
])

const guardado = ref(false)
const alertaGrande = ref(false)
const compraFinalizada = ref(false)


const costoEnvio = ref(45) 
const envioGratis = computed(() => subtotal.value >= 1000)

const totalItems = computed(() => productos.value.reduce((t, p) => t + p.cantidad, 0))
const totalFavoritos = computed(() => productos.value.filter(p => p.favorito).length)
const subtotal = computed(() => productos.value.reduce((t, p) => t + p.precio * p.cantidad, 0))
const impuesto = computed(() => subtotal.value * 0.16)
const totalFinal = computed(() => {
  const envio = envioGratis.value ? 0 : costoEnvio.value
  return subtotal.value + impuesto.value + envio
})


function abrirModalImagen(producto) {
  productoSeleccionado.value = producto
  modalImagen.value = true
}

function seleccionarProducto(producto) {
  productoSeleccionado.value = producto
}

function agregarDesdeModal() {
  if (productoSeleccionado.value) {
    agregarAlCarrito(productoSeleccionado.value)
    modalImagen.value = false
  }
}


function toggleFavorito(producto) {
  producto.favorito = !producto.favorito
  mostrarAlertaFavorito(producto)
  guardarCarrito()
}

function toggleFavoritoModal() {
  if (productoSeleccionado.value) {
    productoSeleccionado.value.favorito = !productoSeleccionado.value.favorito
    mostrarAlertaFavorito(productoSeleccionado.value)
    guardarCarrito()
  }
}

function mostrarAlertaFavorito(producto) {
  favoritoAgregado.value = true
  setTimeout(() => {
    favoritoAgregado.value = false
  }, 2000)
}

function verFavoritos() {
  const favoritos = productos.value.filter(p => p.favorito)
  if (favoritos.length > 0) {
    $q.dialog({
      title: '❤️ Tus Productos Favoritos',
      message: `Tienes ${favoritos.length} productos en favoritos:\n\n${favoritos.map(p => `• ${p.nombre} - $${p.precio}`).join('\n')}`,
      ok: {
        color: 'pink',
        label: 'Cerrar'
      }
    })
  }
}

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

function finalizarCompra() {
  if (totalItems.value === 0) return
  
  compraFinalizada.value = true
  
  setTimeout(() => {
    vaciarCarrito()
    setTimeout(() => {
      compraFinalizada.value = false
    }, 5000)
  }, 1000)
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
      return saved ? { ...p, cantidad: saved.cantidad, agregado: saved.agregado, favorito: saved.favorito || false } : p
    })
  }
})

watch(totalFinal, nuevoTotal => {
  alertaGrande.value = nuevoTotal > 1000
})
</script>

<style scoped>
.heart-notify, .heart-notify-resumen, .badge, .badge-resumen, .badge-favoritos, .badge-favoritos-resumen,
.q-card, .q-btn, .q-icon, .producto-imagen, .cantidad {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.producto-imagen {
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 12px;
  overflow: hidden;
  background: linear-gradient(135deg, #f5f5f5, #e0e0e0);
  padding: 8px;
  flex-shrink: 0;
  position: relative;
  cursor: pointer;
}

.imagen-producto {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 8px;
  transition: transform 0.3s ease;
}

.producto-imagen:hover .imagen-producto {
  transform: scale(1.1);
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
  border-radius: 12px;
}

.producto-imagen:hover .overlay {
  opacity: 1;
}

.zoom-icon {
  color: white;
  font-size: 24px;
}

.card-hover {
  transition: all 0.3s ease;
  cursor: pointer;
}
.card-hover:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0,0,0,0.15);
  border-color: #9C27B0;
}


.pulse {
  animation: pulse 0.3s ease;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.05); }
  100% { transform: scale(1); }
}


.heart-anim {
  transition: transform 0.2s ease;
}
.heart-anim:hover {
  transform: scale(1.2);
  color: #E040FB;
}

.heart-anim-resumen {
  transition: transform 0.2s ease;
  color: #9C27B0;
}
.heart-anim-resumen:hover {
  transform: scale(1.2);
  color: #E040FB;
}


.alertas-container {
  display: flex;
  flex-direction: column;
  gap: 8px;
}


.badge {
  background: linear-gradient(135deg, #FF4081, #E040FB);
  color: white;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  position: absolute;
  top: -5px;
  right: -5px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}

.badge-resumen {
  background: linear-gradient(135deg, #FF4081, #E040FB);
  color: white;
  border-radius: 50%;
  width: 28px;
  height: 28px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 0.9rem;
  position: absolute;
  top: -8px;
  right: -8px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.3);
}


.badge-favoritos {
  background: linear-gradient(135deg, #EC407A, #E91E63);
  color: white;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 0.7rem;
  position: absolute;
  top: -3px;
  left: -3px;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}

.badge-favoritos-resumen {
  background: linear-gradient(135deg, #EC407A, #E91E63);
  color: white;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-size: 0.8rem;
  position: absolute;
  top: -6px;
  left: -6px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.3);
}

.heart-notify, .heart-notify-resumen {
  position: relative;
  display: inline-block;
}


.card-productos {
  border: 2px solid #2196F3 !important;
  border-radius: 12px !important;
  box-shadow: 0 4px 12px rgba(33, 150, 243, 0.2) !important;
}


.card-producto {
  border: 1px solid #E0E0E0 !important;
  border-radius: 8px !important;
  background: linear-gradient(to right, #FFFFFF, #F5F5F5) !important;
  transition: all 0.3s ease !important;
}

.card-producto:hover {
  border-color: #9C27B0 !important;
  background: linear-gradient(to right, #F3E5F5, #E1BEE7) !important;
}

.btn-agregar-morado {
  background: linear-gradient(135deg, #BA68C8, #9C27B0) !important;
  border-radius: 24px !important;
  color: white !important;
  font-weight: bold !important;
}

.btn-finalizar-cool {
  background: linear-gradient(135deg, #00E676, #00C853) !important;
  border-radius: 16px !important;
  color: white !important;
  font-weight: bold !important;
  box-shadow: 0 4px 15px rgba(0, 230, 118, 0.3) !important;
  transition: all 0.3s ease !important;
}
.btn-finalizar-cool:hover:not(.disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 230, 118, 0.4) !important;
}
.btn-finalizar-cool.disabled {
  background: linear-gradient(135deg, #BDBDBD, #9E9E9E) !important;
  box-shadow: none !important;
}

.btn-vaciar-cool {
  background: linear-gradient(135deg, #FF6D00, #FF3D00) !important;
  border-radius: 16px !important;
  color: white !important;
  font-weight: bold !important;
  box-shadow: 0 4px 12px rgba(255, 109, 0, 0.3) !important;
  transition: all 0.3s ease !important;
}
.btn-vaciar-cool:hover:not(.disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 18px rgba(255, 109, 0, 0.4) !important;
}
.btn-vaciar-cool.disabled {
  background: linear-gradient(135deg, #BDBDBD, #9E9E9E) !important;
  box-shadow: none !important;
}

.btn-aumentar {
  background: linear-gradient(135deg, #4CAF50, #2E7D32) !important;
}

.btn-disminuir {
  background: linear-gradient(135deg, #F44336, #C62828) !important;
}

.resumen {
  border: 2px solid #9C27B0 !important;
  border-radius: 16px !important;
  box-shadow: 0 6px 16px rgba(156, 39, 176, 0.2) !important;
  background: linear-gradient(to bottom, #FFFFFF, #F8F9FA) !important;
}

.resumen-detalles {
  background: white;
  border-radius: 12px;
  padding: 20px;
  margin: 10px 0;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.cantidad {
  background: linear-gradient(135deg, #000000, #424242) !important;
  color: white !important;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  box-shadow: 0 2px 5px rgba(0,0,0,0.2);
}

.modal-imagen-card {
  background: rgb(245, 242, 247);
  max-width: 100vw;
  max-height: 100vh;
}

.modal-header {
  border-bottom: 1px solid #e0e0e0;
}

.modal-content {
  display: flex;
  flex-direction: column;
  height: calc(100vh - 50px);
  overflow-y: auto;
}

.imagen-ampliada-container {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #ecf0f3, #d8dcdd);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 20px;
}

.imagen-ampliada {
  max-width: 100%;
  max-height: 400px;
  object-fit: contain;
  border-radius: 8px;
}

.thumbnails-container {
  display: flex;
  gap: 12px;
  justify-content: center;
  flex-wrap: wrap;
  padding: 0 20px;
}

.thumbnail-item {
  width: 60px;
  height: 60px;
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
  border: 2px solid transparent;
  transition: all 0.3s ease;
  background: #f5f5f5;
  display: flex;
  align-items: center;
  justify-content: center;
}

.thumbnail-item.active {
  border-color: #9C27B0;
  box-shadow: 0 0 0 2px rgba(156, 39, 176, 0.3);
}

.thumbnail-item:hover {
  transform: scale(1.1);
}

.thumbnail-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 6px;
}

.producto-info {
  padding: 0 20px;
}

.precio-section {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 10px 0;
}

.text-line-through {
  text-decoration: line-through;
}

.detalles-adicionales {
  background: #f8f9fa;
  padding: 15px;
  border-radius: 8px;
  margin: 15px 0;
}

.botones-modal {
  margin-top: 20px;
}

@media (max-width: 599px) {
  .producto-imagen {
    width: 60px;
    height: 60px;
  }
  
  .cantidad {
    width: 28px;
    height: 28px;
    font-size: 0.9rem;
  }

  .imagen-ampliada-container {
    padding: 10px;
  }

  .thumbnails-container {
    padding: 0 10px;
  }

  .thumbnail-item {
    width: 50px;
    height: 50px;
  }
}

@media (max-width: 399px) {
  .producto-imagen {
    width: 50px;
    height: 50px;
  }
  
  .cantidad {
    width: 24px;
    height: 24px;
    font-size: 0.8rem;
  }

  .thumbnail-item {
    width: 40px;
    height: 40px;
  }
}
</style>