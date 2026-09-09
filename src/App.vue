<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <q-page class="q-pa-md bg-grey-10">

        <div class="q-mb-md bg-black text-amber-14 q-pa-md rounded-borders sombra borde">
          <div class="row items-center justify-between">
            <div>
              <div class="text-h4 text-bold">⚙️ TALLER DON EFRAÍN</div>
              <div class="text-subtitle1 text-grey-4 text-weight-bold">
                SISTEMA DE SERVICIO TÉCNICO
              </div>
            </div>

            <div class="q-mt-sm-device">
              <q-chip v-if="esTecnico" color="positive" text-color="white" icon="admin_panel_settings"
                class="text-bold q-mr-sm">
                TÉCNICO ACTIVO
              </q-chip>

              <q-btn v-if="!esTecnico" label="MODO TÉCNICO" icon="lock" color="amber-14" text-color="black"
                class="text-bold" @click="modalLogin = true" />
              <q-btn v-else label="SALIR" icon="logout" color="negative" class="text-bold" @click="esTecnico = false" />
            </div>
          </div>
        </div>

        <div class="row q-col-gutter-md q-mb-md items-center">
          <div class="col-12 col-md-8">
            <q-btn label="INGRESAR NUEVO EQUIPO" icon="build" color="amber-14" text-color="black"
              class="full-width text-bold" size="lg" @click="abrirNuevo" />
          </div>

          <div class="col-12 col-md-4">
            <div class="bg-grey-9 text-white q-pa-md rounded-borders borde">
              <div class="row justify-between items-center">
                <span class="text-subtitle1 text-bold">
                  EQUIPOS EN TALLER
                </span>
                <q-badge color="orange-10" text-color="white" class="text-subtitle1 q-px-sm" :label="lista.length" />
              </div>

              <q-btn v-if="esTecnico && lista.length > 0" label="BORRAR TODOS LOS PEDIDOS" icon="delete_sweep"
                color="negative" class="text-bold q-mt-sm full-width" dense @click="confirmarLimpiarTodo" />
            </div>
          </div>
        </div>

        <div v-if="lista.length === 0" class="text-center text-grey-5 q-pa-xl bg-grey-9 rounded-borders borde">
          <q-icon name="handyman" size="4rem" color="amber-14" />
          <div class="text-h6 q-mt-sm text-bold">
            No hay trabajos registrados
          </div>
        </div>

        <div v-else class="row q-col-gutter-md">
          <div v-for="(item, index) in lista" :key="item.id" class="col-12 col-sm-6 col-md-4">
            <q-card class="bg-grey-9 text-white borde alto">

              <div :class="{
                'bg-negative': item.estadoPago === 'Pendiente',
                'bg-warning text-black': item.estadoPago === 'Abono',
                'bg-positive': item.estadoPago === 'Pagado'
              }" class="q-pa-sm text-center text-bold text-uppercase">
                PAGO: {{ item.estadoPago || "Sin definir" }}
                <span v-if="item.estadoPago === 'Abono'"> ({{ formatoMoneda(item.abono) }} / Total: {{
                  formatoMoneda(item.precio) }})</span>
                <span v-else-if="item.precio"> - {{ formatoMoneda(item.precio) }}</span>
              </div>

              <q-card-section>
                <div class="row items-center justify-between q-mb-sm">
                  <div class="text-h6 text-bold text-amber-14">
                    {{ item.cliente || "Cliente sin nombre" }}
                  </div>

                  <q-chip dense square text-color="white" :color="colorEstado(item.estadoEquipo)"
                    :icon="iconoEstado(item.estadoEquipo)">
                    {{ item.estadoEquipo }}
                  </q-chip>
                </div>

                <div class="text-body2 text-grey-4 q-mb-sm">
                  Fecha: {{ item.fechaHora }}
                </div>

                <q-separator dark class="q-my-sm" />

                <div class="text-body1"><b>Marca:</b> {{ item.marca || "Sin definir" }}</div>
                <div class="text-body1"><b>Modelo:</b> {{ item.modelo || "Sin definir" }}</div>

                <div class="text-body1 q-mt-xs">
                  <b>Reparaciones / Fallos:</b>
                  <div v-if="Array.isArray(item.tipoReparacion) && item.tipoReparacion.length > 0"
                    class="q-mt-xs row q-gutter-xs">
                    <q-chip v-for="(fallo, fIdx) in item.tipoReparacion" :key="fIdx" dense color="amber-14"
                      text-color="black" class="text-weight-bold">
                      {{ fallo }}
                    </q-chip>
                  </div>
                  <span v-else> {{ item.tipoReparacion || "Sin definir" }}</span>
                </div>

                <div class="text-body1 q-mt-xs"><b>Técnico:</b> {{ item.tecnico || "Sin definir" }}</div>
                <div class="text-body1"><b>Método de pago:</b> {{ item.metodoPago || "Sin definir" }}</div>

                <div v-if="item.estadoPago === 'Abono'" class="text-body1 q-mt-xs text-negative text-bold">
                  <b>Falta por pagar:</b> {{ formatoMoneda((item.precio || 0) - (item.abono || 0)) }}
                </div>

                <div v-if="item.observaciones"
                  class="text-body1 q-mt-sm bg-black text-amber-5 q-pa-sm rounded-borders nota">
                  <b>Observaciones:</b> {{ item.observaciones }}
                </div>

                <div v-if="item.estadoEquipo === 'Entregado'"
                  class="q-mt-md bg-black q-pa-sm rounded-borders text-center">
                  <div class="text-subtitle2 text-bold text-amber-14">
                    ⭐ Calificación del cliente
                  </div>
                  <q-rating v-model="item.calificacion" size="1.8em" color="amber-14" :readonly="esTecnico" :max="5" />
                  <div v-if="!esTecnico" class="text-caption text-grey-4">
                    Haz clic en las estrellas para calificar
                  </div>
                </div>

              </q-card-section>

              <q-separator dark />

              <q-card-actions align="right" class="bg-black">
                <q-btn v-if="item.estadoEquipo !== 'Entregado'" flat color="amber-14" icon="edit" label="Editar"
                  @click="abrirEditar(index)" />

                <q-btn v-if="item.estadoEquipo !== 'Entregado'" flat color="negative" icon="delete" label="Eliminar"
                  @click="confirmarEliminar(index)" />

                <span v-if="item.estadoEquipo === 'Entregado'" class="text-positive text-bold q-pa-sm text-caption">
                  ✓ REGISTRO CERRADO Y ENTREGADO
                </span>
              </q-card-actions>

            </q-card>
          </div>
        </div>

        <q-dialog v-model="modalLogin">
          <q-card class="bg-grey-9 text-white style-modal">
            <q-card-section class="bg-black text-amber-14 borde">
              <div class="text-h6 text-bold">🔐 INGRESO MODO TÉCNICO</div>
            </q-card-section>

            <q-card-section class="q-pa-md">
              <q-input v-model="claveIngresada" :type="isPassword ? 'password' : 'text'" label="Contraseña" dark
                outlined dense color="amber-14" @keyup.enter="verificarClave">
                <template v-slot:append>
                  <q-icon :name="isPassword ? 'visibility_off' : 'visibility'" class="cursor-pointer"
                    @click="isPassword = !isPassword" />
                </template>
              </q-input>

              <div v-if="errorClave" class="text-negative text-caption q-mt-xs text-bold">
                ⚠️ Contraseña incorrecta. LA CONTRASEÑA ES {{ CLAVE_TECNICO }}
              </div>
            </q-card-section>

            <q-card-actions align="right" class="bg-black">
              <q-btn flat label="Cancelar" color="grey-5" v-close-popup />
              <q-btn flat label="Ingresar" color="amber-14" class="text-bold" @click="verificarClave" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <q-dialog v-model="modal">
          <q-card class="bg-grey-9 text-white formulario">

            <q-card-section class="row items-center bg-black text-amber-14 borde">
              <div class="text-h6 text-bold">
                {{ editando ? "MODIFICAR REGISTRO" : "REGISTRAR TRABAJO" }}
              </div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup color="amber-14" />
            </q-card-section>

            <q-card-section>
              <q-form @submit.prevent="guardar">

                <q-input v-model="formulario.cliente" label="Nombre del cliente *" dark outlined color="amber-14"
                  class="q-mb-md" lazy-rules
                  :rules="[val => !!val && val.trim() !== '' || 'El nombre del cliente es obligatorio']" />

                <div class="row q-col-gutter-md">
                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.marca" :options="marcas" label="Marca *" dark outlined
                      color="amber-14" lazy-rules :rules="[val => !!val || 'Selecciona una marca']" />
                  </div>

                  <div class="col-12 col-sm-6">
                    <q-input v-model="formulario.modelo" label="Modelo *" dark outlined color="amber-14" lazy-rules
                      :rules="[val => !!val && val.trim() !== '' || 'Escribe el modelo del equipo']" />
                  </div>
                </div>

                <div class="row q-col-gutter-md q-mt-sm">
                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.tipoReparacion" :options="reparaciones"
                      label="Tipo de reparación (Múltiple) *" multiple use-chips dark outlined color="amber-14"
                      lazy-rules :rules="[val => (val && val.length > 0) || 'Selecciona al menos una reparación']" />
                  </div>

                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.tecnico" :options="tecnicos" label="Técnico *" dark outlined
                      color="amber-14" lazy-rules :rules="[val => !!val || 'Selecciona un técnico']" />
                  </div>
                </div>

                <div class="row q-col-gutter-md q-mt-sm">
                  <div class="col-12 col-sm-6">
                    <q-input v-model.number="formulario.precio" label="Precio *" type="number" dark outlined
                      color="amber-14" lazy-rules :rules="[
                        val => (val !== null && val !== '' && val !== undefined) || 'El precio es obligatorio',
                        val => val >= 0 || 'El precio debe ser un número positivo'
                      ]" />
                  </div>

                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.metodoPago" :options="metodos" label="Método de pago *" dark outlined
                      color="amber-14" lazy-rules :rules="[val => !!val || 'Selecciona un método de pago']" />
                  </div>
                </div>

                <div class="row q-col-gutter-md q-mt-sm">
                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.estadoPago" :options="estadosPago" label="Estado del pago *" dark
                      outlined color="amber-14" lazy-rules :rules="[val => !!val || 'Selecciona el estado del pago']"
                      @update:model-value="validarEstadoEquipoConPago" />
                  </div>

                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.estadoEquipo" :options="opcionesEstadoEquipoFiltradas"
                      label="Estado del equipo *" dark outlined color="amber-14" lazy-rules :rules="[
                        val => !!val || 'Selecciona el estado del equipo',
                        val => (val !== 'Entregado' || formulario.estadoPago === 'Pagado') || 'No se puede entregar si no está totalmente Pagado'
                      ]" />
                    <div v-if="!esTecnico" class="text-caption text-amber-5 q-mt-xs">
                      🔒 Se requiere Modo Técnico para marcar como 'Entregado'.
                    </div>
                    <div v-else-if="formulario.estadoPago !== 'Pagado'" class="text-caption text-negative q-mt-xs">
                      ⚠️ Debe estar completamente 'Pagado' para marcar como 'Entregado'.
                    </div>
                  </div>
                </div>

                <q-input v-if="formulario.estadoPago === 'Abono'" v-model.number="formulario.abono"
                  label="Valor del abono *" type="number" dark outlined color="amber-14" class="q-mt-md" lazy-rules
                  :rules="[
                    val => (val !== null && val !== '' && val !== undefined) || 'Ingresa el valor del abono',
                    val => val > 0 || 'El abono debe ser mayor a 0',
                    val => val < formulario.precio || 'El abono debe ser menor al precio total'
                  ]" />

                <q-input v-model="formulario.observaciones" label="Observaciones" type="textarea" rows="3" dark outlined
                  color="amber-14" class="q-mt-md" />

                <div class="row justify-end q-mt-lg">
                  <q-btn label="Cancelar" color="grey-6" flat v-close-popup class="q-mr-sm" />
                  <q-btn :label="editando ? 'Actualizar' : 'Guardar'" type="submit" color="amber-14" text-color="black"
                    class="text-bold" />
                </div>

              </q-form>
            </q-card-section>

          </q-card>
        </q-dialog>

        <q-dialog v-model="eliminar">
          <q-card class="bg-grey-9 text-white style-modal">
            <q-card-section class="bg-negative text-white text-h6">
              Confirmar eliminación
            </q-card-section>

            <q-card-section class="text-body1">
              ¿Deseas eliminar este registro?
            </q-card-section>

            <q-card-actions align="right" class="bg-black">
              <q-btn flat label="Cancelar" color="grey-5" v-close-popup />
              <q-btn flat label="Eliminar" color="negative" class="text-bold" @click="eliminarRegistro" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <q-dialog v-model="modalLimpiarTodo">
          <q-card class="bg-grey-9 text-white style-modal">
            <q-card-section class="bg-negative text-white text-h6 text-bold">
              ⚠️ ¡ATENCIÓN!
            </q-card-section>

            <q-card-section class="text-body1">
              ¿Estás seguro de que deseas eliminar <b>TODOS</b> los pedidos registrados? Esta acción borrará la lista
              por completo y no se puede deshacer.
            </q-card-section>

            <q-card-actions align="right" class="bg-black">
              <q-btn flat label="Cancelar" color="grey-5" v-close-popup />
              <q-btn flat label="Sí, borrar todo" color="negative" class="text-bold" @click="vaciarLista" />
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

const lista = useLocalStorage("taller_don_efrain", []);
const esTecnico = useLocalStorage("taller_don_efrain_es_tecnico", false);

const CLAVE_TECNICO = "hola";

const modal = ref(false);
const eliminar = ref(false);
const modalLimpiarTodo = ref(false);
const editando = ref(false);
const posicion = ref(null);

const modalLogin = ref(false);
const claveIngresada = ref("");
const isPassword = ref(true);
const errorClave = ref(false);

const formulario = ref({
  cliente: "",
  marca: null,
  modelo: "",
  tipoReparacion: [],
  tecnico: null,
  precio: null,
  metodoPago: null,
  estadoPago: null,
  abono: null,
  estadoEquipo: "Recibido",
  calificacion: 0,
  observaciones: "",
  fechaHora: ""
});

const marcas = [
  "Apple", "Samsung", "Xiaomi", "Motorola",
  "Huawei", "Realme", "OPPO", "Honor"
];

const reparaciones = [
  "Cambio de pantalla", "Cambio de batería", "Cambio de pin de carga",
  "Liberación", "Mantenimiento de software", "Cambio de flex", "Otros"
];

const tecnicos = [
  "Don Efraín", "Omar Leonardo Dangond Rueda", "Javier Esneider Pinto Rodríguez"
];

const metodos = ["Efectivo", "Transferencia", "Tarjeta"];
const estadosPago = ["Pagado", "Pendiente", "Abono"];
const estadosEquipo = ["Recibido", "En reparación", "Listo para entregar", "Entregado"];

function formatoMoneda(valor) {
  if (valor === null || valor === undefined || valor === "" || isNaN(valor)) return "$0";

  let numStr = Math.round(valor).toString();
  let partes = [];

  while (numStr.length > 3) {
    partes.unshift(numStr.slice(-3));
    numStr = numStr.slice(0, -3);
  }
  partes.unshift(numStr);

  if (partes.length >= 3) {
    const millones = partes.slice(0, partes.length - 2).join("'");
    const resto = partes.slice(partes.length - 2).join(".");
    return `$${millones}'${resto}`;
  }

  return `$${partes.join(".")}`;
}

const opcionesEstadoEquipoFiltradas = computed(() => {
  let opciones = [...estadosEquipo];
  if (!esTecnico.value || formulario.value.estadoPago !== "Pagado") {
    opciones = opciones.filter(e => e !== "Entregado");
  }
  return opciones;
});

function validarEstadoEquipoConPago(nuevoEstadoPago) {
  if (nuevoEstadoPago !== "Pagado" && formulario.value.estadoEquipo === "Entregado") {
    formulario.value.estadoEquipo = "Listo para entregar";
  }
}

function verificarClave() {
  if (claveIngresada.value === CLAVE_TECNICO) {
    esTecnico.value = true;
    modalLogin.value = false;
    claveIngresada.value = "";
    errorClave.value = false;
    isPassword.value = true;
  } else {
    errorClave.value = true;
  }
}

function nuevoFormulario() {
  return {
    cliente: "",
    marca: null,
    modelo: "",
    tipoReparacion: [],
    tecnico: null,
    precio: null,
    metodoPago: null,
    estadoPago: null,
    abono: null,
    estadoEquipo: "Recibido",
    calificacion: 0,
    observaciones: "",
    fechaHora: new Date().toLocaleString("es-CO")
  };
}

function abrirNuevo() {
  editando.value = false;
  posicion.value = null;
  formulario.value = nuevoFormulario();
  modal.value = true;
}

function abrirEditar(index) {
  if (lista.value[index].estadoEquipo === "Entregado") return;

  editando.value = true;
  posicion.value = index;

  const datos = JSON.parse(JSON.stringify(lista.value[index]));
  if (typeof datos.tipoReparacion === 'string') {
    datos.tipoReparacion = [datos.tipoReparacion];
  } else if (!Array.isArray(datos.tipoReparacion)) {
    datos.tipoReparacion = [];
  }

  formulario.value = datos;
  modal.value = true;
}

function guardar() {
  if (formulario.value.cliente) {
    formulario.value.cliente = formulario.value.cliente.trim();
  }
  if (formulario.value.modelo) {
    formulario.value.modelo = formulario.value.modelo.trim();
  }

  if (formulario.value.estadoPago !== "Abono") {
    formulario.value.abono = null;
  }

  if (editando.value) {
    lista.value[posicion.value] = {
      ...lista.value[posicion.value],
      ...formulario.value
    };
  } else {
    lista.value.push({
      id: Date.now(),
      ...formulario.value
    });
  }

  modal.value = false;
}

function confirmarEliminar(index) {
  if (lista.value[index].estadoEquipo === "Entregado") return;
  posicion.value = index;
  eliminar.value = true;
}

function eliminarRegistro() {
  if (
    posicion.value !== null &&
    lista.value[posicion.value].estadoEquipo !== "Entregado"
  ) {
    lista.value.splice(posicion.value, 1);
  }
  eliminar.value = false;
  posicion.value = null;
}

function confirmarLimpiarTodo() {
  modalLimpiarTodo.value = true;
}

function vaciarLista() {
  lista.value = [];
  modalLimpiarTodo.value = false;
}

function colorEstado(estado) {
  if (estado === "Entregado") return "grey-8";
  if (estado === "Listo para entregar") return "positive";
  if (estado === "En reparación") return "orange-9";
  return "blue-grey-8";
}

function iconoEstado(estado) {
  if (estado === "Entregado") return "check_circle";
  if (estado === "En reparación") return "engineering";
  return "build";
}
</script>

<style scoped>
.borde {
  border: 1px solid #424242;
  border-bottom: 3px solid #ffb300;
}

.sombra {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.4);
}

.alto {
  height: 100%;
}

.nota {
  border-left: 3px solid #ffb300;
}

.formulario {
  width: 100%;
  max-width: 700px;
}

.style-modal {
  width: 100%;
  max-width: 400px;
}
</style>
