<template>
  <div class="contenedor-principal">
    <!-- CUADRO 1: Logo + Texto -->
    <div class="cuadro cuadro1">
      <div>
        <img class="logo" src="/public/logo.png"></img>
      </div>
      <div class="texto">
        <h1>Organiza rifas en minutos</h1>
        <p>Crea rifas, genera boletas y controla todo el proceso de forma automática y segura.</p>
        <button class="btn-neon" @click="nuevaRifaDesdeCero" >➕ Crear</button>
      </div>
    </div>

    <!-- Mensaje cuando no hay rifa -->
    <div v-if="!rifa?.nombre" class="mensaje-vacio text-center">
      <h2>No hay rifa creada</h2>
      <p>Haz clic en "Crear" para iniciar una nueva rifa.</p>
    </div>

    <!-- Información de la rifa -->
    <div class="cuadro cuadro2" v-if="rifa?.nombre">
      <h2>Información de la Rifa</h2>
      <p><b>Nombre:</b> {{ rifa.nombre }}</p>
      <p><b>Premio:</b> ${{ formatNumero(rifa.premio) }}</p>
      <p><b>Valor Boleta:</b> ${{ formatNumero(rifa.valor) }}</p>
      <p><b>Lotería:</b> {{ rifa.loteria }}</p>
      <p><b>Fecha:</b> {{ rifa.fecha }}</p>
      <p><b>Recaudo Total:</b> ${{ formatNumero(rifa.valor * rifa.cantidad) }}</p>
      <button class="btn-guardar" @click="editarRifa">✏️ Editar</button>
    </div>

    <!-- Acciones -->
    <div class="cuadro cuadro3" v-if="rifa?.nombre">
      <h2>Acciones</h2>
      <button class="btn-neon2 m-1" @click="verEstadisticas">📊 Ver estadísticas</button>
      <button class="btn-neon2 m-1" @click="imprimirEstadisticas">🖨 Imprimir estadísticas</button>
      <button class="btn-neon2 m-1" @click="abrirPersonalizar">🎨 Personalizar</button>
      <button class="btn-neon2 m-1" @click="sorteo">🎉 Sorteo</button>
      <button class="btn-neon2 m-1" @click="eliminarRifa">🗑 Eliminar Rifa</button>
    </div>

    <!-- Boletas -->
    <div class="cuadro cuadro4" v-if="rifa?.nombre && rifa?.cantidad">
      <h2>Boletas</h2>
      <div class="boletas">
        <span
          v-for="n in parseInt(rifa.cantidad)"
          :key="n"
          @click="toggleBoleta(n)"
          class="bolita"
          :class="[estadoBolita(n), (rifa.numeroGanador === n) ? 'ganadora' : ''] "
        >
          {{ n }}
        </span>
      </div>
    </div>

    <!-- Modal Crear/Editar -->
    <div class="modal fade" id="miModal" tabindex="-1" aria-hidden="true" ref="modalRef">
      <div class="modal-dialog">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Configura tu Rifa</h5>
            <button type="button" class="btn-close" @click="confirmarCerrarModal"></button>
          </div>
          <div class="modal-body">
            <div class="formulario">
              <input type="text" class="form-control mb-2" placeholder="Nombre" v-model="nuevaRifa.nombre" />
              <input type="text" class="form-control mb-2" placeholder="Premio" v-model="nuevaRifa.premio" />
              <input type="text" class="form-control mb-2" placeholder="Valor Boleta" v-model="nuevaRifa.valor" />

              <select class="form-select mb-2" v-model="nuevaRifa.loteria">
                <option value="">Selecciona Lotería</option>
                <option>Cruz Roja</option>
                <option>La Perla</option>
                <option>Santander</option>
                <option>Baloto</option>
              </select>

              <select class="form-select mb-2" v-model.number="nuevaRifa.cantidad">
                <option value="">Cantidad de Boletas</option>
                <option value="100">0-99</option>
                <option value="1000">0-999</option>
              </select>

              <input type="date" class="form-control mb-2" v-model="nuevaRifa.fecha" />
            </div>
          </div>
          <div class="modal-footer border-top-0">
            <button class="btn-guardar" @click="guardarRifa">💾 Guardar</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal Registrar Boleta -->
    <div class="modal fade" id="modalBoleta" tabindex="-1" aria-hidden="true" ref="modalBoletaRef">
      <div class="modal-dialog">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Registrar Boleta N° {{ boletaSeleccionada }}</h5>
            <button type="button" class="btn-close" @click="confirmarCerrarModalBoleta"></button>
          </div>
          <div class="modal-body">
            <input type="text" class="form-control mb-2" placeholder="Nombre" v-model="datosBoleta.nombre" />
            <input type="text" class="form-control mb-2" placeholder="Dirección" v-model="datosBoleta.direccion" />
            <input type="text" class="form-control mb-2" placeholder="Teléfono" v-model="datosBoleta.telefono" />

            <select class="form-select mb-2" v-model="datosBoleta.estado">
              <option value="">Seleccione estado</option>
              <option value="Pagada">Pagada</option>
              <option value="Apartada">Apartada</option>
            </select>
          </div>
          <div class="modal-footer">
            <button
              class="btn"
              :class="datosBoleta.estado === 'Pagada' ? 'btn-success' : 'btn-warning'"
              @click="registrarBoleta"
            >
              Registrar
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal Ver Estadísticas -->
    <div class="modal fade" id="modalEstadisticas" tabindex="-1" aria-hidden="true" ref="modalEstadisticasRef">
      <div class="modal-dialog modal-lg">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Estadísticas de la Rifa</h5>
            <button type="button" class="btn-close" @click="cerrarModalEstadisticas"></button>
          </div>
          <div class="modal-body">
            <div ref="tablaEstadisticas">
              <table class="table table-bordered table-dark text-center">
                <thead>
                  <tr>
                    <th>Nombre</th>
                    <th>Dirección</th>
                    <th>Teléfono</th>
                    <th>Fecha Compra</th>
                    <th>Estado</th>
                    <th>N° Boleta</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="detalle in (rifa?.detallesBoletas || [])" :key="detalle.numero">
                    <td>{{ detalle.nombre }}</td>
                    <td>{{ detalle.direccion }}</td>
                    <td>{{ detalle.telefono }}</td>
                    <td>{{ detalle.fecha }}</td>
                    <td>
                      <span :class="detalle.estado === 'Pagada' ? 'text-success' : 'text-warning'">
                        {{ detalle.estado }}
                      </span>
                    </td>
                    <td>{{ detalle.numero }}</td>
                  </tr>
                </tbody>
              </table>
              <div class="resumen-totales mt-3 text-end">
  <p><b>Total Recaudado:</b> ${{ formatNumero(totalRecaudado) }}</p>
  <p><b>Deuda Total:</b> ${{ formatNumero(totalDeuda) }}</p>
</div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal Personalizar -->
    <div class="modal fade" id="modalPersonalizar" tabindex="-1" aria-hidden="true" ref="modalPersonalizarRef">
      <div class="modal-dialog">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Personalizar Estilos</h5>
            <button type="button" class="btn-close" @click="cerrarModalPersonalizar"></button>
          </div>
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label">Color de fondo</label>
              <input type="color" v-model="customStyles.fondo" class="form-control form-control-color" />
            </div>
            <div class="mb-3">
              <label class="form-label">Color de las bolitas</label>
              <input type="color" v-model="customStyles.bolitas" class="form-control form-control-color" />
            </div>
            <div class="mb-3">
              <label class="form-label">Color del borde de los cuadros</label>
              <input type="color" v-model="customStyles.bordeCuadro" class="form-control form-control-color" />
            </div>
            <div class="mb-3">
              <label class="form-label">Color de los botones</label>
              <input type="color" v-model="customStyles.botones" class="form-control form-control-color" />
            </div>
          </div>
          <div class="modal-footer">
            <button class="btn-guardar" @click="guardarPersonalizacion">Guardar</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import Swal from 'sweetalert2'
import html2pdf from 'html2pdf.js'

const rifa = ref({ detallesBoletas: [] })
const nuevaRifa = ref({ nombre: '', premio: '', valor: '', loteria: '', cantidad: '', fecha: '' })
const customStyles = ref({ fondo: '#ff00cc', bolitas: '#3498db', bordeCuadro: '#ff00cc', botones: '#ff0080' })
const boletaSeleccionada = ref(null)
const datosBoleta = ref({ nombre: '', direccion: '', telefono: '', estado: '' })

const modalRef = ref(null)
const modalBoletaRef = ref(null)
const modalEstadisticasRef = ref(null)
const modalPersonalizarRef = ref(null)

onMounted(() => {
  const guardada = localStorage.getItem('rifa')
  if (guardada) {
    try {
      rifa.value = JSON.parse(guardada)
      if (!rifa.value.detallesBoletas) rifa.value.detallesBoletas = []
    } catch {
      rifa.value = { detallesBoletas: [] }
    }
  }

  const estilos = localStorage.getItem('estilos')
  if (estilos) {
    customStyles.value = { ...customStyles.value, ...JSON.parse(estilos) }
  }
  aplicarEstilos()
})

function mostrarAlerta(titulo, mensaje, tipo = 'info') {
  Swal.fire({ title: titulo, text: mensaje, icon: tipo, confirmButtonColor: '#ff0080', background: '#1a1a1a', color: '#fff' })
}

function nuevaRifaDesdeCero() {
  nuevaRifa.value = { nombre: '', premio: '', valor: '', loteria: '', cantidad: '', fecha: '' }
  mostrarModal(modalRef.value)
}

function guardarRifa() {
  // Validar que no haya campos vacíos o solo espacios
  if (!nuevaRifa.value.nombre.trim() || !nuevaRifa.value.premio.trim() || 
      !nuevaRifa.value.valor.trim() || !nuevaRifa.value.loteria.trim() || 
      !nuevaRifa.value.cantidad || !nuevaRifa.value.fecha) {
    mostrarAlerta('Error', 'Rellena todos los campos sin espacios en blanco', 'error')
    return
  }

  // Validar que la fecha sea al menos 8 días después de hoy
  const hoy = new Date()
  const fechaSeleccionada = new Date(nuevaRifa.value.fecha)
  const fechaMinima = new Date(hoy.setDate(hoy.getDate() + 8))

  if (fechaSeleccionada < fechaMinima) {
    mostrarAlerta('Error', 'La fecha debe ser al menos 8 días después de hoy', 'error')
    return
  }

  // Validar que el valor de la boleta no sea mayor al premio
  const premio = parseFloat(nuevaRifa.value.premio)
  const valorBoleta = parseFloat(nuevaRifa.value.valor)

  if (valorBoleta > premio) {
    mostrarAlerta('Error', 'El valor de la boleta no puede ser mayor que el premio', 'error')
    return
  }

  // Guardar si todo está correcto
  rifa.value = { ...nuevaRifa.value, detallesBoletas: [], numeroGanador: null }
  localStorage.setItem('rifa', JSON.stringify(rifa.value))
  ocultarModal(modalRef.value)
  mostrarAlerta('Éxito', 'Rifa creada con éxito', 'success')
}


// Calcula dinero recaudado (Pagadas)
const totalRecaudado = computed(() => {
  return rifa.value.detallesBoletas
    .filter(b => b.estado === 'Pagada')
    .length * (rifa.value.valor || 0)
})

// Calcula deuda total (Apartadas)
const totalDeuda = computed(() => {
  return rifa.value.detallesBoletas
    .filter(b => b.estado === 'Apartada')
    .length * (rifa.value.valor || 0)
})

function editarRifa() {
  nuevaRifa.value = { ...rifa.value }
  mostrarModal(modalRef.value)
}

function toggleBoleta(num) {
  boletaSeleccionada.value = num
  const existente = rifa.value.detallesBoletas.find(b => b.numero === num)
  datosBoleta.value = existente ? { ...existente } : { nombre: '', direccion: '', telefono: '', estado: '' }
  mostrarModal(modalBoletaRef.value)
}

function registrarBoleta() {
  // Validar campos vacíos o solo espacios
  if (
    !datosBoleta.value.nombre.trim() ||
    !datosBoleta.value.direccion.trim() ||
    !datosBoleta.value.telefono.trim() ||
    !datosBoleta.value.estado.trim()
  ) {
    mostrarAlerta('Error', 'Todos los campos son obligatorios y no pueden estar en blanco', 'warning');
    return;
  }

  const idx = rifa.value.detallesBoletas.findIndex(b => b.numero === boletaSeleccionada.value);
  const fechaHoy = new Date().toISOString().split('T')[0];
  const registro = { ...datosBoleta.value, fecha: fechaHoy, numero: boletaSeleccionada.value };

  if (idx === -1) {
    rifa.value.detallesBoletas.push(registro);
  } else {
    rifa.value.detallesBoletas[idx] = registro;
  }

  localStorage.setItem('rifa', JSON.stringify(rifa.value));
  ocultarModal(modalBoletaRef.value);
  mostrarAlerta('Éxito', 'Boleta registrada', 'success');
}

function estadoBolita(num) {
  const detalle = rifa.value.detallesBoletas.find(b => b.numero === num)
  if (!detalle) return ''
  return detalle.estado === 'Pagada' ? 'pagada' : detalle.estado === 'Apartada' ? 'apartada' : ''
}

function sorteo() {
  const pagadas = rifa.value.detallesBoletas.filter(b => b.estado === 'Pagada')
  if (!pagadas.length) {
    mostrarAlerta('Info', 'Debes registrar boletas como Pagadas para el sorteo', 'info')
    return
  }
  const random = Math.floor(Math.random() * pagadas.length)
  rifa.value.numeroGanador = pagadas[random].numero
  localStorage.setItem('rifa', JSON.stringify(rifa.value))
  mostrarAlerta('Ganador', `Número ganador: ${rifa.value.numeroGanador}`, 'success')
}

function eliminarRifa() {
  Swal.fire({
    title: '¿Eliminar rifa?',
    text: 'Esto borrará toda la información registrada',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#d33',
    cancelButtonColor: '#3085d6',
    confirmButtonText: 'Sí, eliminar'
  }).then(result => {
    if (result.isConfirmed) {
      localStorage.removeItem('rifa')
      rifa.value = { detallesBoletas: [] }
      mostrarAlerta('Éxito', 'Rifa eliminada', 'success')
    }
  })
}

function mostrarModal(el) {
  el.style.display = 'block';
  el.classList.add('show');
  document.body.style.overflow = 'hidden';

  // Crear y mostrar el fondo oscuro
  let backdrop = document.createElement('div');
  backdrop.classList.add('modal-backdrop', 'show');
  document.body.appendChild(backdrop);
}

function ocultarModal(el) {
  el.style.display = 'none';
  el.classList.remove('show');
  document.body.style.overflow = 'auto';

  // Eliminar el fondo oscuro
  const backdrop = document.querySelector('.modal-backdrop');
  if (backdrop) backdrop.remove();
}

function confirmarCerrarModal() {
  Swal.fire({
    title: '¿Estás seguro?',
    text: 'Si cierras ahora perderás los datos no guardados de la rifa.',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#d33',
    cancelButtonColor: '#3085d6',
    confirmButtonText: 'Sí, cerrar',
    cancelButtonText: 'Seguir editando'
  }).then(result => {
    if (result.isConfirmed) {
      cerrarModal();
    }
  });
}

function confirmarCerrarModalBoleta() {
  Swal.fire({
    title: '¿Estás seguro?',
    text: 'Si cierras este registro, podrías perder el número y tal vez sea el ganador.',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#d33',
    cancelButtonColor: '#3085d6',
    confirmButtonText: 'Sí, cerrar',
    cancelButtonText: 'No, continuar'
  }).then((result) => {
    if (result.isConfirmed) {
      cerrarModalBoleta();
    }
  });
}

function cerrarModal() { ocultarModal(modalRef.value) }
function cerrarModalBoleta() { ocultarModal(modalBoletaRef.value) }
function cerrarModalEstadisticas() { ocultarModal(modalEstadisticasRef.value) }
function cerrarModalPersonalizar() { ocultarModal(modalPersonalizarRef.value) }

function verEstadisticas() { mostrarModal(modalEstadisticasRef.value) }

function imprimirEstadisticas() {
  const elemento = modalEstadisticasRef.value.querySelector('.modal-body') // solo tabla y totales
  const opciones = {
    margin:       10,
    filename:     `Estadisticas-Rifa-${rifa.value.nombre || 'SinNombre'}.pdf`,
    image:        { type: 'jpeg', quality: 0.98 },
    html2canvas:  { scale: 2 },
    jsPDF:        { unit: 'mm', format: 'a4', orientation: 'portrait' }
  }

  html2pdf().set(opciones).from(elemento).save()
}

function abrirPersonalizar() { mostrarModal(modalPersonalizarRef.value) }

function guardarPersonalizacion() {
  localStorage.setItem('estilos', JSON.stringify(customStyles.value))
  aplicarEstilos()
  ocultarModal(modalPersonalizarRef.value)
  mostrarAlerta('Éxito', 'Estilos guardados', 'success')
}

function aplicarEstilos() {
  document.documentElement.style.setProperty('--color-cuadros', customStyles.value.fondo)
  document.documentElement.style.setProperty('--color-bolitas', customStyles.value.bolitas)
  document.documentElement.style.setProperty('--color-borde-cuadro', customStyles.value.bordeCuadro)
  document.documentElement.style.setProperty('--color-botones', customStyles.value.botones)
}

function formatNumero(num) { return new Intl.NumberFormat('es-CO').format(num || 0) }
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');

:root {
  --color-bolitas: #3498db;
  --color-borde-cuadro: #ff00cc;
  --color-botones: #4db6ac;
  --color-cuadros: #ffffff;
}

body {
  font-family: 'Poppins', sans-serif;
  margin: 0;
  padding: 0;
  min-height: 100vh;
  color: #2c3e50;
  background: #f4f4f4;;
}

.logo{
  align-items: center;
}

/* GRID PRINCIPAL */
.contenedor-principal {
  display: grid;
  grid-template-columns: 2fr 1fr;
  grid-template-rows: auto auto auto;
  gap: 20px;
  max-width: 1200px;
  margin: auto;
  padding: 30px;
}

/* BLOQUE SUPERIOR */
.cuadro1 {
  grid-column: 1 / 3;
  text-align: center;
  padding: 30px;
}

.cuadro1 .logo {
  width: 180px;
  border-radius: 50%;
  margin-bottom: 15px;
}

.cuadro1 h1 {
  font-size: 2rem;
  margin-bottom: 10px;
}

.cuadro1 p {
  font-size: 1rem;
  color: #555;
  margin-bottom: 20px;
}

/* BLOQUE IZQUIERDA (info) y DERECHA (acciones) */
.cuadro1, .cuadro2, .cuadro3, .cuadro4 {
  background: var(--color-cuadros);
  border: 1px solid var(--color-borde-cuadro);
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.05);
  transition: transform 0.2s ease, box-shadow 0.2s ease, background 0.3s ease;
}

.cuadro2:hover, .cuadro3:hover, .cuadro4:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 12px rgba(0,0,0,0.1);
}

.cuadro2 {
  grid-column: 1;
}

.mensaje-vacio {
  grid-column: 1 / 3; /* ocupa todo el ancho del grid */
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 300px; /* altura fija o ajustable según el diseño */
  background: #ffffff;
  border: 1px solid var(--color-borde-cuadro);
  border-radius: 16px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.05);
}

.mensaje-vacio h2 {
  margin-bottom: 10px;
}

.mensaje-vacio p {
  color: #555;
}

.cuadro3 {
  grid-column: 2;
}

/* BLOQUE INFERIOR (boletas) */
.cuadro4 {
  grid-column: 1 / 3;
}

/* BOLETAS */
.cuadro4 .boletas {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(40px, 1fr));
  gap: 8px;
  margin-top: 10px;
}

.bolita {
  width: 38px;
  height: 38px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--color-bolitas);
  border-radius: 50%;
  cursor: pointer;
  border: 2px solid #ffffff;
  font-size: 14px;
  font-weight: 600;
  color: #2c3e50;
  transition: transform 0.2s ease, background 0.3s ease;
}

.bolita:hover {
  transform: scale(1.1);
  background-color: #81c784;
}

.bolita.pagada {
  background: #4caf50;
  color: #fff;
}

.bolita.apartada {
  background: #ffd54f;
  color: #2c3e50;
}

.bolita.ganadora {
  border: 3px solid #ff4081;
  box-shadow: 0 0 10px #ff4081;
}

/* BOTONES */
.btn-neon, .btn-guardar, .btn-neon2 {
  background-color: var(--color-botones);
  color: #fff;
  border: none;
  padding: 10px 18px;
  border-radius: 30px;
  cursor: pointer;
  font-weight: 600;
  transition: background 0.3s ease, transform 0.2s ease;
}

.btn-neon:hover, .btn-guardar:hover, .btn-neon2:hover {
  background-color: #26a69a;
  transform: translateY(-2px);
}

/* TABLA ESTADÍSTICAS */
.table {
  font-size: 0.9rem;
  border-radius: 8px;
  overflow: hidden;
}

.table th {
  background-color: #4db6ac;
  color: #fff;
  text-transform: uppercase;
  font-weight: 600;
}

.table td {
  background: #f9f9f9;
  color: #333;
}

/* FONDO OSCURO CUANDO EL MODAL ESTÁ ABIERTO */
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6); /* oscuro semi-transparente */
  z-index: 1040; /* detrás del modal pero sobre todo lo demás */
  display: none;
}

.modal-backdrop.show {
  display: block;
}

.modal.show .modal-dialog {
  transform: scale(1);
  opacity: 1;
  transition: all 0.3s ease;
}

.modal .modal-dialog {
  transform: scale(0.8);
  opacity: 0;
}

.resumen-totales {
  padding-top: 10px;
  border-top: 2px solid #4db6ac;
  font-size: 1rem;
  color: #2c3e50;
}
.resumen-totales p {
  margin: 4px 0;
}

/* RESPONSIVO */
@media (max-width: 768px) {
  .contenedor-principal {
    grid-template-columns: 1fr;
    padding: 15px;
  }

  .cuadro1, .cuadro2, .cuadro3, .cuadro4 {
    grid-column: 1;
  }

  .cuadro1 h1 {
    font-size: 1.5rem;
  }

  .cuadro4 .boletas {
    grid-template-columns: repeat(auto-fill, minmax(30px, 1fr));
  }
}



</style>
