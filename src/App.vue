<script setup>
import { ref, reactive, watch, computed, onMounted } from "vue";
import Filters from "./components/Filters.vue";
import Presupuesto from "./components/Presupuesto.vue";
import Gasto from "./components/Gasto.vue";
import Modal from "./components/Modal.vue";
import ControlPresupuesto from "./components/ControlPresupuesto.vue";
import iconoNuevoGasto from "./assets/img/nuevo-gasto.svg";
import { generarId } from "./helpers";
const modal = reactive({
  mostrar: false,
  animar: false,
});
const presupuesto = ref(0);
const disponible = ref(0);
const gastado = ref(0);
const filtro = ref(0);
const gasto = reactive({
  nombre: "",
  cantidad: "",
  categoria: "",
  id: null,
  fecha: Date.now(),
});
const gastos = ref([]);
watch(
  gastos,
  () => {
    const totalGastado = gastos.value.reduce(
      (total, gasto) => gasto.cantidad + total,
      0
    );
    gastado.value = totalGastado;
    disponible.value = presupuesto.value - totalGastado;
    localStorage.setItem("gastos", JSON.stringify(gastos.value));
  },

  {
    deep: true,
  }
);
watch(modal, () => {
  if (!modal.mostrar) {
    reiniciarStateGasto();
  }
});
watch(presupuesto, () => {
  localStorage.setItem("presupuesto", presupuesto.value);
});
onMounted(() => {
  const presupuestoStorage = localStorage.getItem("presupuesto");
  if (presupuestoStorage) {
    presupuesto.value = Number(presupuestoStorage);
    disponible.value = Number(presupuestoStorage);
  }
  const gastosStorage = localStorage.getItem("gastos");
  if (gastosStorage) {
    gastos.value = JSON.parse(gastosStorage);
  }
});
const definirPresupuesto = (cantidad) => {
  presupuesto.value = cantidad;
  disponible.value = cantidad;
};
const mostrarModal = () => {
  modal.mostrar = true;
  setTimeout(() => {
    modal.animar = true;
  }, 300);
};
const ocultarModal = () => {
  modal.animar = false;
  setTimeout(() => {
    modal.mostrar = false;
  }, 300);
};
const guardarGasto = () => {
  if (gasto.id) {
    //editando
    const { id } = gasto;
    const i = gastos.value.findIndex((gasto) => gasto.id === id);
    gastos.value[i] = { ...gasto };
  } else {
    //registro nuevo
    gastos.value.push({
      ...gasto,
      id: generarId(),
    });
  }
  ocultarModal();
  //Reiniciar el objeto
  reiniciarStateGasto();
};

const seleccionarGasto = (id) => {
  const gastoEditar = gastos.value.filter((gasto) => gasto.id === id)[0];
  Object.assign(gasto, gastoEditar);
  mostrarModal();
};

const reiniciarStateGasto = () => {
  //Reiniciar el objeto
  Object.assign(gasto, {
    nombre: "",
    cantidad: "",
    categoria: "",
    id: null,
    fecha: Date.now(),
  });
};
const eliminarGasto = () => {
  if (confirm("Eliminar?")) {
    gastos.value = gastos.value.filter(
      (gastoState) => gastoState.id !== gasto.id
    );
    ocultarModal();
  }
};
const gastosFiltrados = computed(() => {
  if (filtro.value) {
    return gastos.value.filter((gasto) => gasto.categoria === filtro.value);
  }
  return gastos.value;
});
const resetApp = () => {
  if (confirm("¿Desear reiniciar presupuesto y gastos?")) {
    gastos.value = [];
    presupuesto.value = 0;
  }
};
</script>

<template>
  <div :class="{ fijar: modal.mostrar }">
    <header>
      <h1>Planificador de Gastos</h1>
      <div class="contenedor-header contenedor sombra">
        <Presupuesto
          v-if="presupuesto === 0"
          @definir-presupuesto="definirPresupuesto"
        />
        <ControlPresupuesto
          v-else
          :presupuesto="presupuesto"
          :disponible="disponible"
          :gastado="gastado"
          @reset-app="resetApp"
        />
      </div>
    </header>

    <main v-if="presupuesto > 0">
      <div class="listado-gastos contenedor">
        <Filters v-model:filtro="filtro" />
        <h2>{{ gastosFiltrados.length > 0 ? "Gastos" : "No hay gastos" }}</h2>
        <Gasto
          v-for="gasto in gastosFiltrados"
          :key="gasto.id"
          :gasto="gasto"
          @seleccionar-gasto="seleccionarGasto"
        />
      </div>
      <div class="crear-gasto">
        <img
          :src="iconoNuevoGasto"
          alt="icono nuevo gasto"
          @click="mostrarModal"
        />
      </div>
      <Modal
        v-if="modal.mostrar"
        @ocultar-modal="ocultarModal"
        @eliminar-gasto="eliminarGasto"
        :modal="modal"
        :disponible="disponible"
        :id="gasto.id"
        @guardar-gasto="guardarGasto"
        v-model:nombre="gasto.nombre"
        v-model:cantidad="gasto.cantidad"
        v-model:categoria="gasto.categoria"
      />
    </main>
  </div>
  <footer>
    <p>/ᐠ｡ꞈ｡ᐟ\ <span> Powered by Moyosoft </span></p>
  </footer>
</template>

<style>
:root {
  --azul: #3b82f6;
  --blanco: #fff;
  --gris-claro: #f5f5f5;
  --gris: #94a3b8;
  --gris-oscuro: #64748b;
  --negro: #000;
}
html {
  font-size: 62.5%;
  box-sizing: border-box;
}
*,
*:before,
*:after {
  box-sizing: inherit;
}
body {
  font-size: 1.6rem;
  font-family: "Lato", sans-serif;
  background-color: var(--gris-claro);
}
h1 {
  font-size: 4rem;
}
h2 {
  font-size: 3rem;
}
.fijar {
  overflow: hidden;
  height: 100vh;
}
header {
  background-color: var(--azul);
}
header h1 {
  padding: 3rem 0;
  margin: 0;
  color: var(--blanco);
  text-align: center;
}
.contenedor {
  width: 90%;
  max-width: 80rem;
  margin: 0 auto;
}
.contenedor-header {
  margin-top: 5rem;
  transform: translateY(5rem);
  padding: 5rem;
}
.sombra {
  box-shadow: 0px 10px 15px -3px rgba(0, 0, 0, 0.1);
  background-color: var(--blanco);
  border-radius: 1.2rem;
  padding: 5rem;
}
.crear-gasto {
  position: fixed;
  bottom: 6rem;
  right: 5rem;
}
.crear-gasto img {
  width: 5rem;
  cursor: pointer;
}
.listado-gastos {
  margin-top: 10rem;
}
.listado-gastos h2 {
  font-weight: bold;
  color: var(--gris-oscuro);
}
footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: var(--azul);
  color: white;
  font-weight: 700;
  text-align: center;
  padding: 0.5px;
}
</style>
