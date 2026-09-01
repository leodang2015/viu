<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <q-page class="q-pa-md bg-grey-10 full-width">
        
        <!-- ENCABEZADO CON CLASE CSS CORTA -->
        <div class="encabezado-principal">
          <div class="text-h4 text-bold">⚙️ TALLER DON EFRAÍN</div>
          <div class="text-subtitle1 text-grey-4 text-weight-bold">SISTEMA DE SERVICIO TÉCNICO</div>
        </div>

        <!-- CONTROLES SUPERIORES -->
        <div class="row q-col-gutter-md q-mb-md items-center">
          <div class="col-12 col-md-5">
            <q-btn 
              label="INGRESAR NUEVO EQUIPO" 
              icon="build" 
              color="amber-14" 
              text-color="black"
              class="full-width text-bold shadow-4" 
              size="lg"
              @click="abrirModalNuevo" 
            />
          </div>

          <div class="col-12 col-md-4">
            <q-input 
              v-model="textoBusqueda" 
              placeholder="Buscar por cliente, marca o modelo..." 
              dark 
              outlined 
              dense 
              color="amber-14"
              clearable
            >
              <template v-slot:prepend>
                <q-icon name="search" color="amber-14" />
              </template>
            </q-input>
          </div>

          <div class="col-12 col-md-3 row justify-between items-center">
            <div class="caja-contador col-8">
              <span class="text-caption text-bold text-grey-3 q-mr-xs">TOTAL:</span>
              <q-badge color="orange-10" text-color="white" class="text-bold" :label="serviciosFiltrados.length" />
            </div>
            
            <q-btn 
              icon="delete_sweep" 
              color="negative" 
              flat 
              round 
              dense 
              :disable="listaServicios.length === 0"
              @click="modalLimpiarTodo = true"
            >
              <q-tooltip>Vaciar todo el registro</q-tooltip>
            </q-btn>
          </div>
        </div>

        <div v-if="serviciosFiltrados.length === 0" class="mensaje-vacio">
          <q-icon name="handyman" size="4rem" color="amber-14" />
          <div class="text-h6 q-mt-sm text-bold">
            {{ listaServicios.length === 0 ? 'Sin equipos registrados.' : 'No se encontraron coincidencias.' }}
          </div>
        </div>

        <!-- REJILLA DE 3 COLUMNAS -->
        <div v-else class="row q-col-gutter-md">
          <div 
            v-for="item in serviciosFiltrados" 
            :key="item.id" 
            class="col-12 col-sm-6 col-md-4"
          >
            <q-card flat class="bg-grey-9 text-white borde-caja full-height column justify-between">
              
              <div>
                <div :class="{
                  'bg-negative': item.estadoPago === 'Pendiente',
                  'bg-warning text-black': item.estadoPago === 'Abono',
                  'bg-positive': item.estadoPago === 'Pagado'
                }" class="q-pa-xs text-center text-weight-bolder text-uppercase text-caption">
                  PAGO: {{ item.estadoPago }} (${{ item.precio }})
                </div>

                <q-card-section class="q-pa-sm">
                  
                  <div class="row items-center justify-between">
                    <div class="text-subtitle1 text-bold text-amber-14 ellipsis" style="max-width: 60%;">
                      {{ item.cliente }}
                    </div>
                    
                    <q-chip 
                      dense 
                      square
                      :color="item.estadoEquipo === 'Entregado' ? 'grey-8' : item.estadoEquipo === 'Listo para entregar' ? 'positive' : 'orange-9'" 
                      text-color="white"
                      :icon="item.estadoEquipo === 'Entregado' ? 'check_circle' : 'engineering'"
                    >
                      {{ item.estadoEquipo }}
                    </q-chip>
                  </div>

                  <div class="text-caption text-grey-5">
                    <b>Fecha:</b> {{ item.fechaHora }}
                  </div>

                  <q-separator dark class="q-my-xs" />

                  <div class="text-body2"><b>Equipo:</b> <span class="text-amber-11">{{ item.marca }} {{ item.modelo }}</span></div>
                  <div class="text-body2"><b>Falla:</b> {{ item.tipoReparacion }}</div>
                  <div class="text-caption text-grey-4"><b>Técnico:</b> {{ item.tecnico }}</div>
                  <div class="text-caption text-grey-4"><b>Pago:</b> {{ item.metodoPago }}</div>

                  <div v-if="item.observaciones" class="text-caption q-mt-xs bg-black text-amber-5 q-pa-xs rounded-borders nota-item">
                    <b>Notas:</b> {{ item.observaciones }}
                  </div>

                  <div v-if="item.estadoEquipo === 'Entregado'" class="q-mt-xs row items-center">
                    <span class="text-caption q-mr-xs text-bold">Calificación:</span>
                    <q-rating v-model="item.calificacion" size="1.2em" color="amber-14" readonly />
                  </div>

                </q-card-section>
              </div>

              <div>
                <q-separator dark />
                <q-card-actions align="right" class="q-pa-xs bg-black">
                  <q-btn flat dense color="amber-14" icon="edit" label="Editar" @click="abrirModalEditar(item.id)" />
                  <q-btn flat dense color="negative" icon="delete" label="Borrar" @click="confirmarBorrado(item.id)" />
                </q-card-actions>
              </div>

            </q-card>
          </div>
        </div>

        <!-- MODAL FORMULARIO -->
        <q-dialog v-model="modalAbierto" persistent>
          <q-card style="width: 100%; max-width: 650px;" class="bg-grey-9 text-white">
            
            <q-card-section class="row items-center bg-black text-amber-14 q-pa-sm borde-arriba">
              <div class="text-subtitle1 text-bold">{{ modoEdicion ? '⚙️ EDITAR DATOS' : '⚙️ REGISTRAR DATOS' }}</div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup color="amber-14" />
            </q-card-section>

            <q-card-section class="q-pa-md">
              <q-form @submit="guardarRegistro" class="q-gutter-y-sm">
                
                <q-input 
                  v-model="formulario.cliente" 
                  label="Nombre del Cliente *" 
                  dark outlined dense 
                  color="amber-14"
                  :rules="[val => !!val || 'Requerido']"
                />

                <div class="row q-col-gutter-xs">
                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.marca" 
                      :options="opcionesMarca" 
                      label="Marca *" 
                      dark outlined dense 
                      color="amber-14"
                      :rules="[val => !!val || 'Requerido']"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.modelo" 
                      :options="opcionesModelo" 
                      label="Modelo *" 
                      dark outlined dense 
                      color="amber-14"
                      :rules="[val => !!val || 'Requerido']"
                    />
                  </div>
                </div>

                <div class="row q-col-gutter-xs">
                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.tipoReparacion" 
                      :options="opcionesReparacion" 
                      label="Tipo Reparación *" 
                      dark outlined dense 
                      color="amber-14"
                      :rules="[val => !!val || 'Requerido']"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.tecnico" 
                      :options="listaTecnicos" 
                      label="Técnico Encargado *" 
                      dark outlined dense 
                      color="amber-14"
                      :rules="[val => !!val || 'Requerido']"
                    />
                  </div>
                </div>

                <div class="row q-col-gutter-xs">
                  <div class="col-12 col-sm-6">
                    <q-input 
                      v-model.number="formulario.precio" 
                      label="Precio ($) *" 
                      type="number" 
                      dark outlined dense 
                      color="amber-14"
                      :rules="[val => val > 0 || 'Inválido']"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.metodoPago" 
                      :options="opcionesMetodoPago" 
                      label="Método Pago *" 
                      dark outlined dense 
                      color="amber-14"
                      :rules="[val => !!val || 'Requerido']"
                    />
                  </div>
                </div>

                <div class="row q-col-gutter-xs">
                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.estadoPago" 
                      :options="opcionesEstadoPago" 
                      label="Estado Pago *" 
                      dark outlined dense 
                      color="amber-14"
                      :rules="[val => !!val || 'Requerido']"
                    />
                  </div>
                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.estadoEquipo" 
                      :options="opcionesEstadoEquipo" 
                      label="Estado Equipo *" 
                      dark outlined dense 
                      color="amber-14"
                      :rules="[val => !!val || 'Requerido']"
                    />
                  </div>
                </div>

                <div v-if="formulario.estadoEquipo === 'Entregado'" class="q-my-xs text-center bg-black q-pa-xs rounded-borders">
                  <div class="text-caption text-bold text-amber-14">Calificación del Cliente:</div>
                  <q-rating v-model="formulario.calificacion" size="2em" color="amber-14" :max="5" />
                </div>

                <q-input 
                  v-model="formulario.observaciones" 
                  label="Observaciones" 
                  type="textarea" 
                  rows="2" 
                  dark outlined dense 
                  color="amber-14"
                />

                <div class="row justify-end q-mt-md">
                  <q-btn label="Cancelar" color="grey-6" flat v-close-popup class="q-mr-xs" />
                  <q-btn :label="modoEdicion ? 'Actualizar' : 'Guardar'" type="submit" color="amber-14" text-color="black" class="text-bold" />
                </div>

              </q-form>
            </q-card-section>
          </q-card>
        </q-dialog>

        <!-- MODAL ELIMINAR UNO -->
        <q-dialog v-model="modalEliminar">
          <q-card style="width: 100%; max-width: 350px;" class="bg-grey-9 text-white">
            <q-card-section class="bg-negative text-white q-pa-sm text-bold">
              Eliminar Registro
            </q-card-section>
            
            <q-card-section class="q-pa-md">
              <div class="text-body2">¿Desea borrar este equipo del taller?</div>
            </q-card-section>

            <q-card-actions align="right" class="bg-black">
              <q-btn flat label="Cancelar" color="grey-5" v-close-popup />
              <q-btn flat label="Eliminar" color="negative" class="text-bold" @click="eliminarDefinitivo" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <!-- MODAL LIMPIAR TODO -->
        <q-dialog v-model="modalLimpiarTodo">
          <q-card style="width: 100%; max-width: 350px;" class="bg-grey-9 text-white">
            <q-card-section class="bg-negative text-white q-pa-sm text-bold">
              ⚠️ Vaciar Registro Completo
            </q-card-section>
            
            <q-card-section class="q-pa-md">
              <div class="text-body2">¿Estás seguro de eliminar TODOS los equipos registrados? Esta acción no se puede deshacer.</div>
            </q-card-section>

            <q-card-actions align="right" class="bg-black">
              <q-btn flat label="Cancelar" color="grey-5" v-close-popup />
              <q-btn flat label="Vaciar Todo" color="negative" class="text-bold" @click="vaciarListaCompleta" />
            </q-card-actions>
          </q-card>
        </q-dialog>

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref, computed } from "vue";
import { useLocalStorage } from "@vueuse/core";

const listaServicios = useLocalStorage("taller_don_efrain_mecanico", []);

const modalAbierto = ref(false);
const modalEliminar = ref(false);
const modalLimpiarTodo = ref(false);
const modoEdicion = ref(false);
const idSeleccionado = ref(null);
const textoBusqueda = ref("");

const formulario = ref({
  cliente: "",
  marca: null,
  modelo: null,
  tipoReparacion: null,
  tecnico: null,
  precio: null,
  metodoPago: null,
  estadoPago: null,
  estadoEquipo: null,
  calificacion: 0,
  observaciones: "",
  fechaHora: ""
});

const opcionesMarca = ["Apple", "Samsung", "Xiaomi", "Motorola", "Huawei", "Realme", "OPPO", "Honor"];
const opcionesModelo = [
  "iPhone 12", "iPhone 13", "iPhone 14", "iPhone 15",
  "Samsung A15", "Samsung A54", "Samsung S23", "Samsung S24",
  "Xiaomi Redmi Note 10", "Xiaomi Redmi Note 12", "Xiaomi 13 Pro",
  "Motorola Moto G84", "Motorola Edge 40",
  "Huawei P60 Pro", "Realme GT Neo 5", "OPPO Reno 10", "Honor Magic 5 Pro"
];
const opcionesReparacion = [
  'Cambio de pantalla', 'Cambio de batería', 'Cambio de pin de carga',
  'Liberación', 'Mantenimiento de software', 'Cambio de flex', 'Otros'
];
const listaTecnicos = [
  'Don Efraín', 'Omar Leonardo Dangond Rueda', 'Javier Esneider Pinto Rodríguez'
];
const opcionesMetodoPago = ['Efectivo', 'Transferencia', 'Tarjeta'];
const opcionesEstadoPago = ['Pagado', 'Pendiente', 'Abono'];
const opcionesEstadoEquipo = ['Recibido', 'En reparación', 'Listo para entregar', 'Entregado'];

const serviciosFiltrados = computed(() => {
  if (!textoBusqueda.value) return listaServicios.value;
  const busqueda = textoBusqueda.value.toLowerCase().trim();
  return listaServicios.value.filter(item => {
    return (
      (item.cliente && item.cliente.toLowerCase().includes(busqueda)) ||
      (item.marca && item.marca.toLowerCase().includes(busqueda)) ||
      (item.modelo && item.modelo.toLowerCase().includes(busqueda))
    );
  });
});

function abrirModalNuevo() {
  modoEdicion.value = false;
  formulario.value = {
    cliente: "",
    marca: null,
    modelo: null,
    tipoReparacion: null,
    tecnico: null,
    precio: null,
    metodoPago: null,
    estadoPago: null,
    estadoEquipo: "Recibido",
    calificacion: 0,
    observaciones: "",
    fechaHora: new Date().toLocaleString('es-CO')
  };
  modalAbierto.value = true;
}

function abrirModalEditar(id) {
  modoEdicion.value = true;
  idSeleccionado.value = id;
  const elemento = listaServicios.value.find(item => item.id === id);
  if (elemento) {
    formulario.value = JSON.parse(JSON.stringify(elemento));
    modalAbierto.value = true;
  }
}

function guardarRegistro() {
  if (modoEdicion.value) {
    const indice = listaServicios.value.findIndex(item => item.id === idSeleccionado.value);
    if (indice !== -1) {
      listaServicios.value[indice] = { ...formulario.value };
    }
  } else {
    listaServicios.value.push({
      id: Date.now(),
      ...formulario.value
    });
  }
  modalAbierto.value = false;
}

function confirmarBorrado(id) {
  idSeleccionado.value = id;
  modalEliminar.value = true;
}

function eliminarDefinitivo() {
  listaServicios.value = listaServicios.value.filter(item => item.id !== idSeleccionado.value);
  modalEliminar.value = false;
}

function vaciarListaCompleta() {
  listaServicios.value = [];
  modalLimpiarTodo.value = false;
}
</script>

<style scoped>
/* AGRUPACIÓN DE CLASES LARGAS EN CLASES LIMPIAS */
.encabezado-principal {
  margin-bottom: 16px;
  text-align: center;
  background-color: #000000;
  color: #ffb300;
  padding: 16px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
  border-bottom: 3px solid #ffb300;
}

.caja-contador {
  background-color: #212121;
  color: #ffffff;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #424242;
  display: flex;
  align-items: center;
}

.mensaje-vacio {
  text-align: center;
  color: #9e9e9e;
  padding: 48px;
  background-color: #212121;
  border-radius: 4px;
}

.borde-arriba {
  border-bottom: 3px solid #ffb300;
}

.borde-caja {
  border: 1px solid #424242;
}

.nota-item {
  border-left: 3px solid #ffb300;
}
</style>