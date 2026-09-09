<template>
  <q-layout view="lHh Lpr lFf" class="bg-grey-2">
    <q-header elevated color="primary">
      <q-toolbar>
        <q-icon name="build" size="30px" class="q-mr-sm" />
        <q-toolbar-title>Taller Don Efraín</q-toolbar-title>
        <q-btn color="secondary" icon="add" label="Nuevo Servicio" @click="abrirModalCrear" />
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="q-pa-md">
        <div v-if="servicios.length === 0" class="text-center q-pa-xl">
          <q-icon name="phonelink_off" size="60px" color="grey" />
          <div class="text-h6 text-grey-7 q-mt-sm">No hay servicios registrados</div>
        </div>

        <div v-else class="row q-col-gutter-md">
          <div v-for="servicio in servicios" :key="servicio.id" class="col-12 col-sm-6 col-md-4">
            <q-card
              flat
              bordered
              :class="{
                'bg-red-1': servicio.estadoPago === 'Pendiente',
                'bg-amber-1': servicio.estadoPago === 'Abono',
                'bg-white': servicio.estadoPago === 'Pagado'
              }"
            >
              <q-item>
                <q-item-section avatar>
                  <q-avatar
                    :color="servicio.estadoEquipo === 'Entregado' ? 'grey' : 'primary'"
                    text-color="white"
                  >
                    <q-icon
                      :name="
                        servicio.estadoEquipo === 'Recibido' ? 'download' :
                        servicio.estadoEquipo === 'En reparación' ? 'build' :
                        servicio.estadoEquipo === 'Listo para entregar' ? 'done' : 'output'
                      "
                    />
                  </q-avatar>
                </q-item-section>
                <q-item-section>
                  <q-item-label class="text-bold">{{ servicio.equipo }}</q-item-label>
                  <q-item-label style="font-size: 20px;" caption>Cliente: {{ servicio.cliente }}</q-item-label>
                </q-item-section>
              </q-item>

              <q-separator />

              <q-card-section class="q-gutter-xs">
                <div><strong>Reparación:</strong> {{ Array.isArray(servicio.tipoReparacion) ? servicio.tipoReparacion.join(', ') : servicio.tipoReparacion }}</div>
                <div v-if="servicio.detalleOtros"><strong>Detalle:</strong> {{ servicio.detalleOtros }}</div>
                <div><strong>Técnico:</strong> {{ servicio.tecnico }}</div>
                <div><strong>Fecha:</strong> {{ servicio.fechaHora }}</div>
                <div><strong>Precio:</strong> ${{ servicio.precio }}</div>
                <div v-if="servicio.estadoPago === 'Abono'"><strong>Abonado:</strong> ${{ servicio.abono }}</div>
                <div><strong>Método de pago:</strong> {{ servicio.metodoPago }}</div>

                <div class="row items-center q-gutter-x-xs">
                  <strong>Estado Pago:</strong>
                  <q-badge style="font-size: 20px;"
                    :color="
                      servicio.estadoPago === 'Pagado' ? 'positive' :
                      servicio.estadoPago === 'Abono' ? 'warning' : 'negative'
                    "
                  >
                    {{ servicio.estadoPago }}
                  </q-badge>
                </div>

                <div class="row items-center q-gutter-x-xs">
                  <strong>Estado Equipo:</strong>
                  <q-badge color="info" style="font-size: 20px;">
                    {{ servicio.estadoEquipo }}
                  </q-badge>
                </div>

                <div v-if="servicio.estadoEquipo === 'Entregado'" class="row items-center">
                  <strong class="q-mr-xs">Calificación:</strong>
                  <q-rating v-model="servicio.calificacion" readonly size="1em" color="amber" />
                </div>

                <div v-if="servicio.observaciones" class="text-caption text-italic bg-grey-3 q-pa-xs rounded-borders q-mt-sm" style="font-size: 20px;">
                  Observaciones: {{ servicio.observaciones }}
                </div>
              </q-card-section>

              <q-separator />

              <q-card-actions align="right">
                <q-btn flat icon="edit" color="primary" label="Editar" @click="abrirModalEditar(servicio)" />
                <q-btn flat icon="delete" color="negative" label="Eliminar" @click="pedirConfirmacionEliminar(servicio.id)" />
              </q-card-actions>
            </q-card>
          </div>
        </div>
      </q-page>
    </q-page-container>

    <q-dialog v-model="modalAbierto" persistent>
      <q-card style="width: 500px; max-width: 90vw;">
        <q-card-section class="bg-primary text-white">
          <div class="text-h6">{{ idEditando ? 'Editar Servicio' : 'Nuevo Servicio' }}</div>
        </q-card-section>

        <q-card-section>
          <q-form @submit="guardarServicio" class="q-gutter-sm">
            <q-input
              v-model="cliente"
              label="Nombre del cliente"
              outlined
              dense
              :rules="[val => !!val || 'El cliente es obligatorio']"
            />

            <q-input
              v-model="equipo"
              label="Marca y modelo del equipo"
              outlined
              dense
              :rules="[val => !!val || 'El equipo es obligatorio']"
            />

            <q-field
              label="Tipo de reparación"
              borderless
              dense
              stack-label
              :model-value="tipoReparacion"
              :rules="[val => val && val.length > 0 || 'Selecciona al menos una reparación']"
            >
              <template #control>
                <div class="full-width column items-start q-gutter-xs q-py-xs">
                  <q-radio
                    v-for="opcion in opcionesReparacion"
                    :key="opcion"
                    :model-value="tipoReparacion.includes(opcion)"
                    :label="opcion"
                    :val="true"
                    @update:model-value="alternarTipoReparacion(opcion)"
                  />
                </div>
              </template>
            </q-field>

            <q-input
              v-if="tipoReparacion.includes('Otros')"
              v-model="detalleOtros"
              label="Detalle de otros servicios"
              outlined
              dense
              input-class="text-left"
              :rules="[val => !!val || 'Describe el servicio seleccionado']"
            />

            <q-select
              v-model="tecnico"
              :options="opcionesTecnicos"
              label="Técnico que atendió"
              outlined
              dense
              input-class="text-left"
              :rules="[val => !!val || 'Selecciona el técnico']"
            />

            <q-input
              v-model="precio"
              type="number"
              label="Precio cobrado"
              outlined
              dense
              :rules="[
                val => val !== '' && val !== null || 'El precio es obligatorio',
                val => val >= 0 || 'Debe ser un valor positivo'
              ]"
            />

            <q-select
              v-model="metodoPago"
              :options="opcionesMetodo"
              label="Método de pago"
              outlined
              dense
              input-class="text-left"
              :rules="[val => !!val || 'Selecciona método de pago']"
            />

            <q-select
              v-model="estadoPago"
              :options="opcionesEstadoPago"
              label="Estado del pago"
              outlined
              dense
              input-class="text-left"
              :rules="[val => !!val || 'Selecciona estado de pago']"
            />

            <q-input
              v-if="estadoPago === 'Abono'"
              v-model="abono"
              type="number"
              label="Cantidad abonada"
              outlined
              dense
              input-class="text-left"
              :rules="[
                val => val !== '' && val !== null || 'La cantidad abonada es obligatoria',
                val => Number(val) > 0 || 'Debe ser mayor que cero',
                val => Number(val) <= Number(precio) || 'No puede superar el precio'
              ]"
            />

            <q-select
              v-model="estadoEquipo"
              :options="opcionesEstadoEquipo"
              :option-disable="opcion => opcion === 'Entregado' && estadoPago !== 'Pagado'"
              label="Estado del equipo"
              outlined
              dense
              input-class="text-right"
              :rules="[val => !!val || 'Selecciona estado del equipo']"
            />

            <div v-if="estadoEquipo === 'Entregado'" class="q-my-sm">
              <div>Calificación del cliente:</div>
              <q-rating v-model="calificacion" size="1.8em" color="amber" icon="star" />
            </div>

            <q-input
              v-model="observaciones"
              type="textarea"
              label="Observaciones (opcional)"
              outlined
              dense
              rows="2"
            />

            <div class="row justify-end q-gutter-sm q-mt-md">
              <q-btn label="Cancelar" color="grey" flat v-close-popup />
              <q-btn type="submit" label="Guardar" color="primary" />
            </div>
          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <q-dialog v-model="modalEliminarAbierto" persistent>
      <q-card style="width: 350px;">
        <q-card-section class="row items-center">
          <q-avatar icon="warning" color="negative" text-color="white" class="q-mr-sm" />
          <span class="text-subtitle1">¿Deseas eliminar este servicio?</span>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Cancelar" color="grey" v-close-popup />
          <q-btn flat label="Eliminar" color="negative" @click="confirmarEliminar" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script setup>
import { ref, watch } from 'vue'

const datosGuardados = localStorage.getItem('servicios_taller')
const servicios = ref(datosGuardados ? JSON.parse(datosGuardados) : [])

const modalAbierto = ref(false)
const modalEliminarAbierto = ref(false)
const idEditando = ref(null)
const idParaEliminar = ref(null)

const cliente = ref('')
const equipo = ref('')
const tipoReparacion = ref([])
const detalleOtros = ref('')
const tecnico = ref('')
const precio = ref('')
const abono = ref('')
const metodoPago = ref('Efectivo')
const estadoPago = ref('Pendiente')
const estadoEquipo = ref('Recibido')
const calificacion = ref(5)
const observaciones = ref('')

const opcionesReparacion = [
  'Cambio de pantalla',
  'Cambio de batería',
  'Cambio de pin de carga',
  'Liberación',
  'Mantenimiento de software',
  'Cambio de flex',
  'Otros'
]
const opcionesTecnicos = ['Don Efraín', 'Técnico 1', 'Técnico 2']
const opcionesMetodo = ['Efectivo', 'Transferencia', 'Tarjeta']
const opcionesEstadoPago = ['Pagado', 'Pendiente', 'Abono']
const opcionesEstadoEquipo = ['Recibido', 'En reparación', 'Listo para entregar', 'Entregado']

function alternarTipoReparacion(opcion) {
  tipoReparacion.value = tipoReparacion.value.includes(opcion)
    ? tipoReparacion.value.filter(item => item !== opcion)
    : [...tipoReparacion.value, opcion]

  if (opcion === 'Otros' && !tipoReparacion.value.includes('Otros')) {
    detalleOtros.value = ''
  }
}

function guardarEnLocalStorage() {
  localStorage.setItem('servicios_taller', JSON.stringify(servicios.value))
}

function abrirModalCrear() {
  idEditando.value = null
  cliente.value = ''
  equipo.value = ''
  tipoReparacion.value = []
  detalleOtros.value = ''
  tecnico.value = ''
  precio.value = ''
  abono.value = ''
  metodoPago.value = 'Efectivo'
  estadoPago.value = 'Pendiente'
  estadoEquipo.value = 'Recibido'
  calificacion.value = 5
  observaciones.value = ''
  modalAbierto.value = true
}

function abrirModalEditar(servicio) {
  idEditando.value = servicio.id
  cliente.value = servicio.cliente
  equipo.value = servicio.equipo
  tipoReparacion.value = Array.isArray(servicio.tipoReparacion)
    ? servicio.tipoReparacion
    : servicio.tipoReparacion ? [servicio.tipoReparacion] : []
  detalleOtros.value = servicio.detalleOtros || ''
  tecnico.value = servicio.tecnico
  precio.value = servicio.precio
  abono.value = servicio.abono || ''
  metodoPago.value = servicio.metodoPago
  estadoEquipo.value = servicio.estadoEquipo
  estadoPago.value = servicio.estadoPago
  calificacion.value = servicio.calificacion
  observaciones.value = servicio.observaciones
  modalAbierto.value = true
}

function guardarServicio() {
  if (idEditando.value !== null) {
    for (let i = 0; i < servicios.value.length; i++) {
      if (servicios.value[i].id === idEditando.value) {
        servicios.value[i].cliente = cliente.value
        servicios.value[i].equipo = equipo.value
        servicios.value[i].tipoReparacion = tipoReparacion.value
        servicios.value[i].detalleOtros = tipoReparacion.value.includes('Otros') ? detalleOtros.value : ''
        servicios.value[i].tecnico = tecnico.value
        servicios.value[i].precio = precio.value
        servicios.value[i].abono = estadoPago.value === 'Abono' ? abono.value : ''
        servicios.value[i].metodoPago = metodoPago.value
        servicios.value[i].estadoPago = estadoPago.value
        servicios.value[i].estadoEquipo = estadoEquipo.value
        servicios.value[i].calificacion = calificacion.value
        servicios.value[i].observaciones = observaciones.value
        break
      }
    }
  } else {
    const nuevoServicio = {
      id: Date.now(),
      fechaHora: new Date().toLocaleString(),
      cliente: cliente.value,
      equipo: equipo.value,
      tipoReparacion: tipoReparacion.value,
      detalleOtros: tipoReparacion.value.includes('Otros') ? detalleOtros.value : '',
      tecnico: tecnico.value,
      precio: precio.value,
      abono: estadoPago.value === 'Abono' ? abono.value : '',
      metodoPago: metodoPago.value,
      estadoPago: estadoPago.value,
      estadoEquipo: estadoEquipo.value,
      calificacion: calificacion.value,
      observaciones: observaciones.value
    }
    servicios.value.push(nuevoServicio)
  }
  guardarEnLocalStorage()
  modalAbierto.value = false
}

watch(estadoPago, nuevoEstadoPago => {
  if (nuevoEstadoPago !== 'Pagado' && estadoEquipo.value === 'Entregado') {
    estadoEquipo.value = 'Listo para entregar'
  }
})

function pedirConfirmacionEliminar(id) {
  idParaEliminar.value = id
  modalEliminarAbierto.value = true
}

function confirmarEliminar() {
  servicios.value = servicios.value.filter(s => s.id !== idParaEliminar.value)
  guardarEnLocalStorage()
  idParaEliminar.value = null
}
</script>