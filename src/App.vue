<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <q-page class="q-pa-md bg-dark">

        <div class="q-mb-md bg-grey-9 text-amber-5 q-pa-md rounded-borders sombra borde">
          <div class="row items-center justify-between">
            <div>
              <div class="text-h4 text-bold">⚙️ TALLER DON EFRAÍN</div>
              <div class="text-subtitle1 text-grey-4 text-weight-bold">
                SISTEMA DE SERVICIO TÉCNICO Y CONTROL DE CLIENTE
                <span class="q-ml-md text-amber-4">({{ esTecnico ? 'MODO TÉCNICO' : 'MODO CLIENTE' }})</span>
              </div>
            </div>

            <div class="q-mt-sm-device row q-gutter-sm items-center">
              <q-btn v-if="!esTecnico" label="LOGIN TÉCNICO" icon="lock" color="amber-7" text-color="black"
                class="text-bold" @click="modalLogin = true" />
              <q-btn v-else label="SALIR TÉCNICO" icon="logout" color="negative" class="text-bold" @click="cerrarSesionTecnico" />
            </div>
          </div>
        </div>

        <div class="row q-col-gutter-md q-mb-md items-center">
          <div class="col-12 col-md-8">
            <q-btn label="INGRESAR NUEVO EQUIPO" icon="build" color="amber-7" text-color="black"
              class="full-width text-bold" size="lg" @click="abrirNuevo" />
          </div>

          <div class="col-12 col-md-4">
            <div class="bg-grey-9 text-white q-pa-md rounded-borders borde">
              <div class="row justify-between items-center">
                <span class="text-subtitle1 text-bold">
                  EQUIPOS EN TALLER
                </span>
                <q-badge color="amber-9" text-color="black" class="text-subtitle1 text-bold q-px-sm" :label="lista.length" />
              </div>

              <q-btn v-if="esTecnico && lista.length > 0" label="BORRAR TODOS LOS PEDIDOS" icon="delete_sweep"
                color="negative" class="text-bold q-mt-sm full-width" dense @click="confirmarLimpiarTodo" />
            </div>
          </div>
        </div>

        <div v-if="lista.length === 0" class="text-center text-grey-5 q-pa-xl bg-grey-9 rounded-borders borde">
          <q-icon name="handyman" size="4rem" color="amber-7" />
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
                'bg-positive text-black': item.estadoPago === 'Pagado'
              }" class="q-pa-sm text-center text-bold text-uppercase">
                PAGO: {{ item.estadoPago || "Sin definir" }}
                <span v-if="item.estadoPago === 'Abono'">
                  ({{ formatoMoneda(item.abono) }} / TOTAL: {{ formatoMoneda(item.precio) }} - FALTA: {{
                    formatoMoneda((item.precio || 0) - (item.abono || 0)) }})
                </span>
                <span v-else-if="item.precio"> - {{ formatoMoneda(item.precio) }}</span>
              </div>

              <q-card-section>
                <div class="row items-center justify-between q-mb-sm">
                  <div class="text-h6 text-bold text-amber-5">
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
                    <q-chip v-for="(fallo, fIdx) in item.tipoReparacion" :key="fIdx" dense
                      text-color="black" class="chip-personalizado">
                      {{ fallo }}
                    </q-chip>
                  </div>
                  <span v-else> {{ item.tipoReparacion || "Sin definir" }}</span>
                </div>

                <div v-if="item.mejorasExtra && item.mejorasExtra.length > 0" class="q-mt-md bg-grey-10 q-pa-sm rounded-borders borde">
                  <div class="text-subtitle2 text-amber-5 text-bold">
                    🔧 Mejoras adicionales propuestas por el técnico:
                  </div>
                  <div v-for="(mejora, mIdx) in item.mejorasExtra" :key="mIdx" class="q-mt-xs text-body2">
                    • {{ mejora.descripcion }} ({{ formatoMoneda(mejora.costo) }})
                    <q-chip dense :color="mejora.estado === 'Aceptado' ? 'positive' : mejora.estado === 'Rechazado' ? 'negative' : 'warning'" text-color="white" class="q-ml-sm text-bold">
                      {{ mejora.estado }}
                    </q-chip>
                  </div>
                </div>

                <!-- PANEL DE CLIENTE PARA APROBAR MEJORAS (Corregido: visible para cualquier estado si hay mejoras) -->
                <div v-if="!esTecnico && item.mejorasExtra && item.mejorasExtra.length > 0" class="q-mt-md bg-grey-10 q-pa-sm rounded-borders borde">
                  <div class="text-subtitle2 text-amber-5 text-bold">
                    👤 Panel de Aprobación (Vista Cliente)
                  </div>
                  <div class="text-caption text-grey-3 q-mb-sm">
                    Active el interruptor para aceptar la mejora adicional. Nota: Requiere pago total para poder activarlo.
                  </div>
                  
                  <div v-for="(mejora, mIdx) in item.mejorasExtra" :key="mIdx" class="row items-center justify-between q-mt-xs bg-dark q-pa-xs rounded-borders">
                    <span class="text-body2">{{ mejora.descripcion }} - {{ formatoMoneda(mejora.costo) }}</span>
                    <q-toggle
                      :model-value="mejora.estado === 'Aceptado'"
                      @update:model-value="(val) => cambiarEstadoSwitchMejora(index, mIdx, val)"
                      :disable="item.estadoPago !== 'Pagado'"
                      color="positive"
                      dark
                      keep-color
                      :label="mejora.estado === 'Aceptado' ? 'Aceptado' : 'Rechazado'"
                    />
                  </div>

                  <div v-if="item.estadoPago !== 'Pagado'" class="text-negative text-caption q-mt-xs text-bold">
                    ⚠️ Los interruptores están desactivados porque el equipo no tiene el pago completamente realizado ("Pagado").
                  </div>
                </div>

                <div v-if="!esTecnico && item.estadoEquipo === 'Listo para entregar'" class="q-mt-md bg-grey-10 q-pa-sm rounded-borders borde">
                  <div class="text-subtitle2 text-amber-5 text-bold">
                    📦 Aviso de Recogida
                  </div>
                  <div class="text-caption text-grey-3 q-mb-sm">
                    Active este interruptor para confirmar al taller que pasará a recoger su equipo.
                  </div>
                  <div class="row items-center justify-between bg-dark q-pa-xs rounded-borders">
                    <span class="text-body2 text-bold">¿Voy a recogerlo?</span>
                    <q-toggle
                      v-model="item.voyARecogerlo"
                      color="positive"
                      dark
                      keep-color
                      :label="item.voyARecogerlo ? 'Sí, voy en camino' : 'No confirmado'"
                    />
                  </div>
                </div>

                <div class="text-body1 q-mt-xs"><b>Técnico:</b> {{ item.tecnico || "Sin definir" }}</div>
                <div class="text-body1"><b>Método de pago:</b> {{ item.metodoPago || "Sin definir" }}</div>

                <div v-if="item.estadoPago === 'Abono'" class="q-mt-sm q-pa-xs rounded-borders bg-grey-10 text-white">
                  <div class="text-body1 text-warning">
                    <b>Abonado:</b> {{ formatoMoneda(item.abono) }}
                  </div>
                  <div class="text-body1 text-negative text-bold">
                    <b>Falta por pagar:</b> {{ formatoMoneda((item.precio || 0) - (item.abono || 0)) }}
                  </div>
                </div>

                <div v-if="item.observaciones"
                  class="text-body1 q-mt-sm bg-grey-10 text-amber-3 q-pa-sm rounded-borders nota">
                  <b>Observaciones:</b> {{ item.observaciones }}
                </div>

                <div v-if="item.estadoEquipo === 'Entregado'"
                  class="q-mt-md bg-grey-10 q-pa-sm rounded-borders text-center">
                  <div class="text-subtitle2 text-bold text-amber-5">
                    ⭐ Calificación del cliente
                  </div>
                  <q-rating v-model="item.calificacion" size="1.8em" color="amber-5" :readonly="esTecnico || item.calificacion > 0" :max="5" />
                  <div v-if="!esTecnico && item.calificacion === 0" class="text-caption text-grey-4">
                    Haz clic en las estrellas para calificar
                  </div>
                  <div v-if="esTecnico" class="text-caption text-grey-5 q-mt-xs">
                    🔒 Cierre sesión de técnico para calificar como cliente.
                  </div>
                </div>

              </q-card-section>

              <q-separator dark />

              <q-card-actions v-if="item.estadoEquipo !== 'Entregado'" align="right" class="bg-grey-10">
                <q-btn v-if="esTecnico" flat color="amber-5" icon="edit" label="Editar / Mejorar" @click="abrirEditar(index)" />
                <q-btn v-if="esTecnico" flat color="negative" icon="delete" label="Eliminar" @click="confirmarEliminar(index)" />
                <span v-if="!esTecnico" class="text-amber-5 text-caption text-bold q-pa-xs">
                  Modo Cliente: Solo lectura / Aprobaciones
                </span>
              </q-card-actions>
              <div v-else class="bg-grey-10 text-center q-pa-sm">
                <span class="text-positive text-bold text-caption">
                  ✓ REGISTRO CERRADO Y ENTREGADO
                </span>
              </div>

            </q-card>
          </div>
        </div>

        <q-dialog v-model="modalLogin">
          <q-card class="bg-grey-9 text-white style-modal">
            <q-card-section class="bg-grey-10 text-amber-5 borde">
              <div class="text-h6 text-bold">🔐 INGRESO MODO TÉCNICO</div>
            </q-card-section>

            <q-card-section class="q-pa-md">
              <q-input v-model="claveIngresada" :type="isPassword ? 'password' : 'text'" label="Contraseña" dark
                outlined dense color="amber-5" @keyup.enter="verificarClave">
                <template v-slot:append>
                  <q-icon :name="isPassword ? 'visibility_off' : 'visibility'" class="cursor-pointer"
                    @click="isPassword = !isPassword" />
                </template>
              </q-input>

              <div v-if="errorClave" class="text-negative text-caption q-mt-xs text-bold">
                ⚠️ Contraseña incorrecta. (Contraseña: hola)
              </div>
            </q-card-section>

            <q-card-actions align="right" class="bg-grey-10">
              <q-btn flat label="Cancelar" color="grey-5" v-close-popup />
              <q-btn flat label="Ingresar" color="amber-5" class="text-bold" @click="verificarClave" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <q-dialog v-model="modal">
          <q-card class="bg-grey-9 text-white formulario">

            <q-card-section class="row items-center bg-grey-10 text-amber-5 borde">
              <div class="text-h6 text-bold">
                {{ editando ? "MODIFICAR / AGREGAR MEJORAS TÉCNICAS" : "REGISTRAR NUEVO EQUIPO" }}
              </div>
              <q-space />
              <q-btn icon="close" flat round dense v-close-popup color="amber-5" />
            </q-card-section>

            <q-card-section class="q-pa-lg">
              <q-form @submit.prevent="guardar">

                <q-input v-model="formulario.cliente" label="Nombre del cliente *" dark outlined color="amber-5"
                  class="q-mb-md input-grande" lazy-rules
                  :rules="[val => !!val && val.trim() !== '' || 'El nombre del cliente es obligatorio']" />

                <div class="row q-col-gutter-md">
                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.marca" 
                      :options="marcasFiltradas" 
                      label="Marca de celular *" 
                      dark 
                      outlined
                      color="amber-5" 
                      use-input
                      input-debounce="0"
                      behavior="menu"
                      @filter="filtrarMarcas"
                      @focus="filtrarMarcas('', (cb) => cb())"
                      lazy-rules 
                      :rules="[val => !!val || 'Selecciona una marca']" 
                      @update:model-value="formulario.modelo = ''"
                      class="input-grande"
                    />
                  </div>

                  <div class="col-12 col-sm-6">
                    <q-select 
                      v-model="formulario.modelo" 
                      :options="modelosFiltrados" 
                      label="Modelo de celular *" 
                      dark 
                      outlined 
                      color="amber-5" 
                      use-input
                      input-debounce="0"
                      behavior="menu"
                      @filter="filtrarModelos"
                      @focus="filtrarModelos('', (cb) => cb())"
                      lazy-rules
                      :rules="[val => !!val && String(val).trim() !== '' || 'Selecciona o escribe el modelo']" 
                      class="input-grande"
                    />
                  </div>
                </div>

                <div class="row q-col-gutter-md q-mt-sm">
                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.tipoReparacion" :options="reparaciones"
                      label="Tipo de reparación (Múltiple) *" multiple use-chips dark outlined color="amber-5"
                      class="chips-compactos input-grande custom-select-reparacion" option-value="value" option-label="label" emit-value map-options
                      lazy-rules :rules="[val => (val && val.length > 0) || 'Selecciona al menos una reparación']"
                      @update:model-value="calcularPrecioAutomatico">
                      <template v-slot:option="{ itemProps, opt }">
                        <q-item v-bind="itemProps" class="text-white bg-grey-9">
                          <q-item-section>
                            <q-item-label>{{ opt.label }}</q-item-label>
                          </q-item-section>
                          <q-item-section side>
                            <span class="text-amber-5 text-bold">{{ opt.precioTexto }}</span>
                          </q-item-section>
                        </q-item>
                      </template>
                    </q-select>
                  </div>

                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.tecnico" :options="tecnicos" label="Técnico *" dark outlined
                      color="amber-5" lazy-rules :rules="[val => !!val || 'Selecciona un técnico']" class="input-grande" />
                  </div>
                </div>

                <div v-if="formulario.tipoReparacion && formulario.tipoReparacion.includes('Otros')" class="q-mt-md">
                  <q-input v-model="formulario.otroReparacion" label="Especificar otra reparación *" dark outlined color="amber-5" class="input-grande" lazy-rules :rules="[val => !formulario.tipoReparacion.includes('Otros') || (!!val && val.trim() !== '') || 'Debe especificar el detalle de otros']" />
                </div>

                <div v-if="formulario.tipoReparacion && formulario.tipoReparacion.includes('Otros')" class="q-mt-md">
                  <q-input :model-value="formatoPrecioOtroInput" label="Precio de la reparación personalizada *" prefix="$" dark outlined color="amber-5" class="input-grande" @update:model-value="actualizarPrecioOtro" />
                </div>

                <div v-if="editando" class="q-mt-md bg-grey-10 q-pa-md rounded-borders borde">
                  <div class="text-subtitle2 text-amber-5 text-bold q-mb-sm">
                    ➕ Agregar Nuevas Mejoras o Elementos Encontrados (Técnico)
                  </div>
                  
                  <div class="row q-col-gutter-sm">
                    <div class="col-12">
                      <q-select 
                        v-model="nuevaMejoraSeleccion" 
                        :options="reparacionesDisponiblesParaMejora" 
                        label="Seleccionar mejoras adicionales *" 
                        multiple 
                        use-chips
                        dark 
                        outlined 
                        dense 
                        color="amber-5"
                        class="chips-compactos input-grande"
                        option-value="value" 
                        option-label="label" 
                        emit-value 
                        map-options
                        @update:model-value="actualizarMontoYCamposMejoras"
                      >
                        <template v-slot:option="{ itemProps, opt }">
                          <q-item v-bind="itemProps" class="text-white bg-grey-9">
                            <q-item-section>
                              <q-item-label>{{ opt.label }}</q-item-label>
                            </q-item-section>
                            <q-item-section side>
                              <span class="text-amber-5 text-bold">{{ opt.precioTexto }}</span>
                            </q-item-section>
                          </q-item>
                        </template>
                      </q-select>
                    </div>
                  </div>

                  <div v-if="nuevaMejoraSeleccion && nuevaMejoraSeleccion.includes('Otros')" class="q-mt-md">
                    <q-input v-model="nuevaMejoraTextoPersonalizado" label="Especificar otra mejora adicional *" dark outlined dense color="amber-5" class="input-grande" />
                  </div>

                  <div v-if="nuevaMejoraSeleccion && nuevaMejoraSeleccion.includes('Otros')" class="q-mt-md">
                    <q-input :model-value="formatoCostoMejoraOtroInput" label="Costo de la mejora personalizada *" prefix="$" dark outlined dense color="amber-5" class="input-grande" @update:model-value="actualizarCostoMejoraOtro" />
                  </div>

                  <div class="row justify-end q-mt-md">
                    <q-btn label="Añadir Mejoras Seleccionadas" color="amber-7" text-color="black" class="text-bold" @click="agregarMejorasExtra" />
                  </div>
                </div>

                <div class="row q-col-gutter-md q-mt-sm">
                  <div class="col-12 col-sm-6">
                    <q-input :model-value="formatoPrecioInput" label="Precio *" prefix="$" dark outlined
                      color="amber-5" class="input-grande" lazy-rules @update:model-value="actualizarPrecio" :rules="[
                        val => (formulario.precio !== null && formulario.precio !== '' && formulario.precio !== undefined) || 'El precio es obligatorio',
                        val => formulario.precio >= 0 || 'El precio debe ser un número positivo'
                      ]" />
                  </div>

                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.metodoPago" :options="metodos" label="Método de pago *" dark outlined
                      color="amber-5" class="input-grande" lazy-rules :rules="[val => !!val || 'Selecciona un método de pago']" />
                  </div>
                </div>

                <div class="row q-col-gutter-md q-mt-sm">
                  <div class="col-12 col-sm-6">
                    <q-select v-model="formulario.estadoPago" :options="estadosPago" label="Estado del pago *" dark
                      outlined color="amber-5" class="input-grande" lazy-rules :rules="[val => !!val || 'Selecciona el estado del pago']"
                      @update:model-value="validarEstadoEquipoConPago" />
                  </div>

                  <div class="col-12 col-sm-6">
                    <q-select v-if="!editando" model-value="Recibido" label="Estado del equipo" dark outlined color="amber-5" class="input-grande" disable />
                    <q-select v-else v-model="formulario.estadoEquipo" :options="opcionesEstadoEquipoFiltradas"
                      label="Estado del equipo *" dark outlined color="amber-5" class="input-grande" lazy-rules :rules="[
                        val => !!val || 'Selecciona el estado del equipo',
                        val => (val !== 'Entregado' || formulario.estadoPago === 'Pagado') || 'No se puede entregar si no está totalmente Pagado',
                        val => (val !== 'Listo para entregar' || permitirDarListo) || 'El cliente aún no ha revisado/aceptado todas las mejoras propuestas.',
                        val => !esFaseAnterior(val) || 'No está permitido regresar el equipo a una fase anterior.',
                        val => (val !== 'Entregado' || formulario.voyARecogerlo) || 'El cliente debe activar el switch de que va a recoger el equipo antes de marcarlo como Entregado.'
                      ]" />
                    <div v-if="!editando" class="text-caption text-amber-4 q-mt-xs">
                      ℹ️ Todo equipo ingresa inicialmente como 'Recibido'.
                    </div>
                    <div v-else-if="!permitirDarListo" class="text-caption text-negative q-mt-xs">
                      ⚠️ Hay mejoras pendientes por revisar por parte del cliente. No se puede poner 'Listo para entregar'.
                    </div>
                  </div>
                </div>

                <div v-if="formulario.estadoPago === 'Abono'" class="q-mt-md">
                  <q-input :model-value="formatoAbonoInput" label="Valor del abono *" prefix="$" dark outlined
                    color="amber-5" class="input-grande" lazy-rules @update:model-value="actualizarAbono" :rules="[
                      val => (formulario.abono !== null && formulario.abono !== '' && formulario.abono !== undefined) || 'Ingresa el valor del abono',
                      val => formulario.abono > 0 || 'El abono debe ser mayor a 0',
                      val => formulario.abono < formulario.precio || 'El abono debe ser menor al precio total'
                    ]" />
                </div>

                <q-input v-model="formulario.observaciones" label="Observaciones" type="textarea" rows="3" dark outlined
                  color="amber-5" class="q-mt-md input-grande" />

                <div class="row justify-end q-mt-lg q-gutter-sm">
                  <q-btn label="Cancelar" color="grey-6" flat v-close-popup size="md" />
                  <q-btn :label="editando ? 'Actualizar' : 'Guardar'" type="submit" color="amber-7" text-color="black"
                    class="text-bold px-md" size="md" />
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

            <q-card-actions align="right" class="bg-grey-10">
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
              ¿Estás seguro de que deseas eliminar <b>TODOS</b> los pedidos registrados?
            </q-card-section>

            <q-card-actions align="right" class="bg-grey-10">
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

const lista = useLocalStorage("taller_don_efrain_v2", []);
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

const nuevaMejoraSeleccion = ref([]);
const nuevaMejoraCostoOtro = ref(0);
const nuevaMejoraTextoPersonalizado = ref("");

const formulario = ref({
  cliente: "",
  marca: null,
  modelo: "",
  tipoReparacion: [],
  otroReparacion: "",
  precioOtro: 0,
  mejorasExtra: [],
  tecnico: null,
  precio: null,
  metodoPago: null,
  estadoPago: null,
  abono: null,
  estadoEquipo: "Recibido",
  voyARecogerlo: false,
  calificacion: 0,
  observaciones: "",
  fechaHora: ""
});

const marcas = [
  "Apple", "Samsung", "Xiaomi", "Motorola",
  "Huawei", "Realme", "OPPO", "Honor", "Vivo", "ZTE", "Infinix", "Tecno"
];

const marcasFiltradas = ref(marcas);
function filtrarMarcas(val, update) {
  update(() => {
    if (val === '') {
      marcasFiltradas.value = marcas;
    } else {
      const needle = val.toLowerCase();
      marcasFiltradas.value = marcas.filter(v => v.toLowerCase().indexOf(needle) > -1);
    }
  });
}

const modelosPorMarca = {
  "Apple": ["iPhone 11", "iPhone 12", "iPhone 13", "iPhone 14", "iPhone 15", "iPhone 16", "iPhone SE", "iPhone X", "iPhone 8"],
  "Samsung": ["Galaxy A14", "Galaxy A24", "Galaxy A34", "Galaxy A54", "Galaxy A55", "Galaxy S23", "Galaxy S24", "Galaxy A04s", "Galaxy A15", "Galaxy S22"],
  "Xiaomi": ["Redmi Note 11", "Redmi Note 12", "Redmi Note 13", "Poco X5", "Poco X6", "Redmi 12", "Redmi 13C", "Poco M5", "Xiaomi 13"],
  "Motorola": ["Moto G22", "Moto G32", "Moto G54", "Moto Edge 40", "Moto G14", "Moto G24", "Moto G84"],
  "Huawei": ["P30 Lite", "Nova 9", "Y9 Prime", "P50 Pro", "Y7 Prime", "Nova 10"],
  "Realme": ["Realme 9", "Realme 10", "Realme C55", "Realme C35", "Realme 11 Pro"],
  "OPPO": ["Reno 7", "Reno 8", "A57", "A78", "A58", "Reno 10"],
  "Honor": ["Honor X7", "Honor X8", "Honor 90", "Honor Magic5", "Honor X5"],
  "Vivo": ["Vivo Y16", "Vivo Y27", "Vivo V25", "Vivo Y36"],
  "ZTE": ["Blade V30", "Blade A52", "Blade V40"],
  "Infinix": ["Hot 30", "Note 30", "Smart 7"],
  "Tecno": ["Spark 10", "Camon 20", "Pova 5"]
};

const modelosFiltrados = ref([]);
function filtrarModelos(val, update) {
  update(() => {
    const listaBase = (formulario.value.marca && modelosPorMarca[formulario.value.marca]) 
      ? modelosPorMarca[formulario.value.marca] 
      : [];

    if (val === '') {
      modelosFiltrados.value = listaBase;
    } else {
      const needle = val.toLowerCase();
      const filtrados = listaBase.filter(v => v.toLowerCase().indexOf(needle) > -1);
      if (filtrados.length === 0 && val.trim() !== '') {
        modelosFiltrados.value = [val]; 
      } else {
        modelosFiltrados.value = filtrados;
      }
    }
  });
}

const preciosReparaciones = {
  "Cambio de pantalla": 150000,
  "Cambio de batería": 80000,
  "Cambio de pin de carga": 50000,
  "Liberación": 40000,
  "Mantenimiento de software": 35000,
  "Cambio de flex": 60000
};

const reparaciones = [
  { label: "Cambio de pantalla", value: "Cambio de pantalla", precioTexto: "$150.000" },
  { label: "Cambio de batería", value: "Cambio de batería", precioTexto: "$80.000" },
  { label: "Cambio de pin de carga", value: "Cambio de pin de carga", precioTexto: "$50.000" },
  { label: "Liberación", value: "Liberación", precioTexto: "$40.000" },
  { label: "Mantenimiento de software", value: "Mantenimiento de software", precioTexto: "$35.000" },
  { label: "Cambio de flex", value: "Cambio de flex", precioTexto: "$60.000" },
  { label: "Otros", value: "Otros", precioTexto: "Variable" }
];

const reparacionesDisponiblesParaMejora = computed(() => {
  const seleccionadasPrincipal = formulario.value.tipoReparacion || [];
  return reparaciones.filter(rep => !seleccionadasPrincipal.includes(rep.value));
});

const tecnicos = [
  "Don Efraín", "Omar Leonardo Dangond Rueda", "Javier Esneider Pinto Rodríguez"
];

const metodos = ["Efectivo", "Transferencia", "Tarjeta"];
const estadosPago = ["Pagado", "Pendiente", "Abono"];
const estadosEquipo = ["Recibido", "En reparación", "Listo para entregar", "Entregado"];

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

function cerrarSesionTecnico() {
  esTecnico.value = false;
}

function formatearNumeroTexto(valor) {
  if (valor === null || valor === undefined || valor === "" || isNaN(valor)) return "";
  let numStr = Math.round(valor).toString();
  let partes = [];
  while (numStr.length > 3) {
    partes.unshift(numStr.slice(-3));
    numStr = numStr.slice(0, -3);
  }
  partes.unshift(numStr);
  if (partes.length >= 3) {
    const millones = partes.slice(0, partes.length - 2).join(",");
    const resto = partes.slice(partes.length - 2).join(".");
    return `${millones},${resto}`;
  }
  return partes.join(".");
}

const formatoPrecioInput = computed(() => formatearNumeroTexto(formulario.value.precio));
const formatoAbonoInput = computed(() => formatearNumeroTexto(formulario.value.abono));
const formatoPrecioOtroInput = computed(() => formatearNumeroTexto(formulario.value.precioOtro));
const formatoCostoMejoraOtroInput = computed(() => formatearNumeroTexto(nuevaMejoraCostoOtro.value));

function actualizarPrecio(val) {
  const soloNumeros = val ? val.replace(/\D/g, "") : "";
  formulario.value.precio = soloNumeros ? parseInt(soloNumeros, 10) : null;
}

function actualizarAbono(val) {
  const soloNumeros = val ? val.replace(/\D/g, "") : "";
  formulario.value.abono = soloNumeros ? parseInt(soloNumeros, 10) : null;
}

function actualizarPrecioOtro(val) {
  const soloNumeros = val ? val.replace(/\D/g, "") : "";
  formulario.value.precioOtro = soloNumeros ? parseInt(soloNumeros, 10) : 0;
  calcularPrecioAutomatico();
}

function actualizarCostoMejoraOtro(val) {
  const soloNumeros = val ? val.replace(/\D/g, "") : "";
  nuevaMejoraCostoOtro.value = soloNumeros ? parseInt(soloNumeros, 10) : 0;
}

function actualizarMontoYCamposMejoras() {
  if (!nuevaMejoraSeleccion.value || !nuevaMejoraSeleccion.value.includes('Otros')) {
    nuevaMejoraTextoPersonalizado.value = "";
    nuevaMejoraCostoOtro.value = 0;
  }
}

function calcularPrecioAutomatico() {
  let total = 0;
  if (formulario.value.tipoReparacion && Array.isArray(formulario.value.tipoReparacion)) {
    formulario.value.tipoReparacion.forEach(rep => {
      if (preciosReparaciones[rep]) {
        total += preciosReparaciones[rep];
      }
    });
  }
  if (formulario.value.tipoReparacion && formulario.value.tipoReparacion.includes('Otros')) {
    total += Number(formulario.value.precioOtro || 0);
  }
  if (formulario.value.mejorasExtra) {
    formulario.value.mejorasExtra.forEach(m => {
      total += Number(m.costo || 0);
    });
  }
  formulario.value.precio = total > 0 ? total : null;
}

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
    const millones = partes.slice(0, partes.length - 2).join(",");
    const resto = partes.slice(partes.length - 2).join(".");
    return `$${millones},${resto}`;
  }
  return `$${partes.join(".")}`;
}

const permitirDarListo = computed(() => {
  if (!formulario.value.mejorasExtra || formulario.value.mejorasExtra.length === 0) return true;
  const hayPendientes = formulario.value.mejorasExtra.some(m => m.estado === "Pendiente");
  return !hayPendientes;
});

function esFaseAnterior(estadoObjetivo) {
  if (posicion.value === null || posicion.value === undefined) return false;
  const estadoOriginal = lista.value[posicion.value]?.estadoEquipo;
  const ordenFases = {
    "Recibido": 1,
    "En reparación": 2,
    "Listo para entregar": 3,
    "Entregado": 4
  };
  return (ordenFases[estadoObjetivo] || 0) < (ordenFases[estadoOriginal] || 0);
}

const opcionesEstadoEquipoFiltradas = computed(() => {
  if (posicion.value === null || posicion.value === undefined) return estadosEquipo;
  const estadoOriginal = lista.value[posicion.value]?.estadoEquipo || "Recibido";
  const ordenFases = {
    "Recibido": 1,
    "En reparación": 2,
    "Listo para entregar": 3,
    "Entregado": 4
  };
  const nivelActual = ordenFases[estadoOriginal] || 1;

  let opciones = estadosEquipo.filter(e => (ordenFases[e] || 0) >= nivelActual);

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

function agregarMejorasExtra() {
  if (!nuevaMejoraSeleccion.value || nuevaMejoraSeleccion.value.length === 0) return;

  if (!formulario.value.mejorasExtra) {
    formulario.value.mejorasExtra = [];
  }

  nuevaMejoraSeleccion.value.forEach(sel => {
    if (sel === "Otros") {
      if (nuevaMejoraTextoPersonalizado.value && nuevaMejoraTextoPersonalizado.value.trim() !== "") {
        formulario.value.mejorasExtra.push({
          descripcion: nuevaMejoraTextoPersonalizado.value.trim(),
          costo: Number(nuevaMejoraCostoOtro.value || 0),
          estado: "Pendiente"
        });
      }
    } else {
      let costoPredefinido = preciosReparaciones[sel] || 0;
      formulario.value.mejorasExtra.push({
        descripcion: sel,
        costo: costoPredefinido,
        estado: "Pendiente"
      });
    }
  });

  calcularPrecioAutomatico();
  nuevaMejoraSeleccion.value = [];
  nuevaMejoraCostoOtro.value = 0;
  nuevaMejoraTextoPersonalizado.value = "";
}

function cambiarEstadoSwitchMejora(indexItem, indexMejora, valorSwitch) {
  const item = lista.value[indexItem];
  if (item && item.mejorasExtra && item.mejorasExtra[indexMejora]) {
    const nuevoEstado = valorSwitch ? "Aceptado" : "Rechazado";
    const estadoAnterior = item.mejorasExtra[indexMejora].estado;
    const costoMejora = Number(item.mejorasExtra[indexMejora].costo || 0);

    item.mejorasExtra[indexMejora].estado = nuevoEstado;

    if (estadoAnterior === "Aceptado" && nuevoEstado === "Rechazado") {
      item.precio = Math.max(0, (item.precio || 0) - costoMejora);
    } else if (estadoAnterior === "Rechazado" && nuevoEstado === "Aceptado") {
      item.precio = (item.precio || 0) + costoMejora;
    }

    lista.value = [...lista.value];
  }
}

function nuevoFormulario() {
  return {
    cliente: "",
    marca: null,
    modelo: "",
    tipoReparacion: [],
    otroReparacion: "",
    precioOtro: 0,
    mejorasExtra: [],
    tecnico: null,
    precio: null,
    metodoPago: null,
    estadoPago: null,
    abono: null,
    estadoEquipo: "Recibido",
    voyARecogerlo: false,
    calificacion: 0,
    observaciones: "",
    fechaHora: new Date().toLocaleString("es-CO")
  };
}

function abrirNuevo() {
  editando.value = false;
  posicion.value = null;
  formulario.value = nuevoFormulario();
  formulario.value.estadoEquipo = "Recibido";
  modal.value = true;
}

function abrirEditar(index) {
  if (lista.value[index].estadoEquipo === "Entregado") return;

  editando.value = true;
  posicion.value = index;
  nuevaMejoraSeleccion.value = [];
  nuevaMejoraCostoOtro.value = 0;
  nuevaMejoraTextoPersonalizado.value = "";

  const datos = JSON.parse(JSON.stringify(lista.value[index]));
  if (!datos.mejorasExtra) datos.mejorasExtra = [];
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

  if (!editando.value) {
    formulario.value.estadoEquipo = "Recibido";
  }

  if (formulario.value.estadoPago !== "Abono") {
    formulario.value.abono = null;
  }

  let reparacionesFinales = [...formulario.value.tipoReparacion];
  if (reparacionesFinales.includes("Otros")) {
    reparacionesFinales = reparacionesFinales.filter(r => r !== "Otros");
    if (formulario.value.otroReparacion && formulario.value.otroReparacion.trim() !== "") {
      reparacionesFinales.push(formulario.value.otroReparacion.trim());
    }
  }

  const datosAGuardar = {
    ...formulario.value,
    tipoReparacion: reparacionesFinales
  };
  delete datosAGuardar.otroReparacion;
  delete datosAGuardar.precioOtro;

  if (editando.value) {
    lista.value[posicion.value] = {
      ...lista.value[posicion.value],
      ...datosAGuardar
    };
  } else {
    lista.value.push({
      id: Date.now(),
      ...datosAGuardar
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
  if (posicion.value !== null && lista.value[posicion.value].estadoEquipo !== "Entregado") {
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
  border: 1px solid #3d3d3d;
  border-bottom: 3px solid #ffb300;
  box-shadow: 0 0 10px rgba(255, 179, 0, 0.15);
}

.sombra {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.6), 0 0 15px rgba(255, 179, 0, 0.08);
}

.alto {
  height: 100%;
}

.nota {
  border-left: 3px solid #ffb300;
}

.formulario {
  width: 100%;
  max-width: 750px;
}

.style-modal {
  width: 100%;
  max-width: 400px;
}

.chip-personalizado,
.custom-select-reparacion :deep(.q-chip),
.chips-compactos :deep(.q-chip),
.q-dialog :deep(.q-chip) {
  background-color: #ffffff !important;
  color: #000000 !important;
  font-weight: normal !important;
  font-size: 1.1rem !important;
  border: 1px solid #cccccc !important;
  box-shadow: none !important;
}

.chip-personalizado *,
.custom-select-reparacion :deep(.q-chip) *,
.chips-compactos :deep(.q-chip) *,
.q-dialog :deep(.q-chip) * {
  color: #000000 !important;
}

.chips-compactos :deep(.q-field__control) {
  min-height: 56px !important;
  height: auto !important;
  padding-bottom: 6px !important;
  padding-top: 6px !important;
}

.input-grande :deep(.q-field__native),
.input-grande :deep(.q-field__input),
.input-grande :deep(.q-field__label) {
  font-size: 1.05rem !important;
}
</style>
