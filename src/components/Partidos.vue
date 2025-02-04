<template>
  <div class="max-w-3xl mx-auto p-6 bg-white shadow-md rounded-lg">
    <h2 class="text-2xl text-cyan-500 font-bold mb-6">Partidos disponibles</h2>

    <!-- Componente Loader mientras los partidos están cargando -->
    <Loader v-if="cargando" />

    <div v-else>
      <!-- Botón para obtener ubicación -->
      <button @click="obtenerUbicacionUsuario"
        class="mb-4 border border-blue-500 text-white bg-blue-500 rounded p-2 hover:bg-transparent hover:text-black transition">
        Obtener ubicación
      </button>
      <!-- Mostrar ubicación del usuario -->
      <p v-if="ubicacionUsuario" class="text-gray-600 flex items-center">
        <i class="fas fa-map-marker-alt mr-2"></i>
        Tu ubicación: {{ ubicacionUsuario.direccion }}
      </p>

      <!-- Filtros -->
      <div class="mt-4 space-y-4">
        <!-- Filtro por deporte -->
        <label class="block text-lg font-semibold text-gray-700">Filtro por deporte:</label>
        <select v-model="filtroDeporte"
          class="w-full border border-gray-300 rounded-md p-2 focus:outline-none focus:ring-2 focus:ring-blue-500">
          <option value="">Todos</option>
          <option value="futbol">Fútbol</option>
          <option value="basquet">Básquet</option>
          <option value="tenis">Tenis</option>
        </select>

        <!-- Filtro por cercanía -->
        <label class="block text-lg font-semibold text-gray-700">Filtro por cercanía:</label>
        <input type="checkbox" v-model="filtroCercania" />
        <span>Mostrar solo partidos cercanos</span>
      </div>

      <!-- Listado de Partidos -->
      <div v-if="partidosFiltrados.length" class="mt-6 space-y-4">
        <div v-for="partido in partidosFiltrados" :key="partido.id" class="border p-4 rounded shadow-sm">
          <!-- Imagen del partido -->
          <img :src="partido.imagen" alt="Imagen del partido" class="w-full h-72 object-cover rounded mb-2" />
          <!-- Detalles del partido -->
          <h3 class="text-xl font-semibold">{{ partido.nombre }}</h3>

          <!-- Organizado por -->
          <p class="text-gray-600 flex items-center">
            <i class="fas fa-user-circle mr-2"></i>
            Organizado por<span class="font-medium">: {{ partido.organizadoPor }}</span>
          </p>

          <!-- Fecha y hora -->
          <p class="text-gray-500 flex items-center">
            <i class="fas fa-calendar-alt mr-3"></i>
            {{ formatoFechaHora(partido.fechaHora) }}
          </p>

          <!-- Precio -->
          <p class="text-gray-500 flex items-center">
            <i class="fas fa-dollar-sign mr-4"></i>
            {{ partido.precio }}
          </p>

          <!-- Deporte -->
          <p class="text-gray-500 flex items-center">
            <i class="fas fa-basketball-ball mr-3" v-if="partido.deporte === 'basquet'"></i>
            <i class="fas fa-futbol mr-2" v-if="partido.deporte === 'futbol'"></i>
            <img src="/img/pelota-de-tenis.png" alt="Pelota de Tenis" class="mr-2" v-if="partido.deporte === 'tenis'" />
            <span>{{ partido.deporte }}</span>
          </p>

          <!-- Ubicación -->
          <p class="text-gray-500 flex items-center">
            <i class="fas fa-map-marker-alt mr-3"></i>
            {{ abreviarDireccion(partido.direccion) }}
          </p>

          <!-- Lugares disponibles -->
          <p class="text-gray-500 flex items-center">
            <i class="fas fa-users mr-1"></i>
            Lugares disponibles: {{ partido.lugaresDisponibles }}
          </p>

          <!-- Botón para unirse al partido -->
          <button v-if="partido.lugaresDisponibles >= 1" @click="unirseAPartido(partido)"
            class="mt-2 border border-cyan-400 text-white bg-cyan-500 rounded p-2 hover:bg-transparent hover:text-black transition px-4 py-2  mr-2">
            Unirse al partido
          </button>

          <p class="text-cyan-400 font-semibold" v-else>No hay lugares disponibles :c</p>
        </div>
      </div>

      <!-- Mensaje cuando no hay partidos -->
      <p v-else class="text-gray-500 mt-6">No hay partidos disponibles.</p>

      <!-- Modal de confirmación de unión -->
      <div v-if="modalVisible" class="fixed inset-0 flex items-center justify-center bg-gray-800 bg-opacity-50">
        <div class="bg-white p-6 rounded-lg max-w-md">
          <h3 class="text-xl font-semibold mb-2">Te has unido al partido</h3>
          <!-- Detalles del partido -->
          <p class="text-gray-500">Nombre: {{ partidoUnido.nombre }}</p>
          <p class="text-gray-500">Organizado por: {{ partidoUnido.organizadoPor }}</p>
          <p class="text-gray-500">Fecha y hora: {{ formatoFechaHora(partidoUnido.fechaHora) }}</p>
          <p class="text-gray-500">Precio: ${{ partidoUnido.precio }}</p>

          <!-- Botones -->
          <div class="mt-4">
            <button @click="abrirModalPago" class="bg-blue-500 text-white rounded p-2 mr-2">Pagar ahora</button>
            <button @click="cerrarModal" class="bg-gray-500 text-white rounded p-2">Cerrar</button>
          </div>
        </div>
      </div>

      <!-- Modal de pago -->
      <div v-if="modalPagoVisible" class="fixed inset-0 flex items-center justify-center bg-gray-800 bg-opacity-50">
        <div class="bg-white p-6 rounded-lg max-w-md">
          <h3 class="text-xl font-semibold mb-4">Formulario de Pago</h3>
          <form @submit.prevent="procesarPago">
            <div class="mb-4">
              <label class="block text-gray-700">Nombre del titular:</label>
              <input type="text" v-model="formPago.nombreTitular" required
                class="w-full border border-gray-300 rounded p-2" />
            </div>
            <div class="mb-4">
              <label class="block text-gray-700">Número de tarjeta:</label>
              <input type="text" v-model="formPago.numeroTarjeta" required
                class="w-full border border-gray-300 rounded p-2" />
            </div>
            <div class="flex gap-4">
              <div class="mb-4">
                <label class="block text-gray-700">Fecha de expiración:</label>
                <input type="text" v-model="formPago.fechaExpiracion" placeholder="MM/YY" required
                  class="w-full border border-gray-300 rounded p-2" />
              </div>
              <div class="mb-4">
                <label class="block text-gray-700">CVV:</label>
                <input type="text" v-model="formPago.cvv" required class="w-full border border-gray-300 rounded p-2" />
              </div>
            </div>
            <div class="mt-4">
              <button type="submit" class="bg-green-500 text-white rounded p-2 mr-2">Confirmar pago</button>
              <button @click="cerrarModalPago" class="bg-gray-500 text-white rounded p-2">Cancelar</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>

</template>

<script>
import { ref, computed, onMounted } from 'vue';
import { getFirestore, collection, onSnapshot, updateDoc, doc } from "firebase/firestore";
import Swal from 'sweetalert2';
import Loader from './Loader.vue'; 

export default {
  components: {
    Loader,
  },
  setup() {
    const db = getFirestore();
    const partidos = ref([]); // Lista de partidos obtenidos desde Firebase
    const ubicacionUsuario = ref(null); // Ubicación actual del usuario
    const cargando = ref(true); // Estado de carga
    const filtroDeporte = ref(""); // Filtro por deporte
    const filtroCercania = ref(false); // Filtro por cercanía
    const modalVisible = ref(false); // Control de visibilidad del modal
    const partidoUnido = ref(null); // Partido al que el usuario se unió

    // Cargar los partidos desde Firebase
    const cargarPartidos = () => {
      const partidosCollection = collection(db, "partidos");
      onSnapshot(partidosCollection, (snapshot) => {
        partidos.value = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
        cargando.value = false; 
      });
    };

    // Obtener la ubicación del usuario
    const obtenerUbicacionUsuario = () => {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          async (position) => {
            ubicacionUsuario.value = {
              lat: position.coords.latitude,
              lng: position.coords.longitude,
            };
            await obtenerDireccion(ubicacionUsuario.value.lat, ubicacionUsuario.value.lng);
          },
          (error) => {
            console.error("Error al obtener ubicación:", error);
            Swal.fire('Error', 'No se pudo obtener la ubicación. Activa el GPS.', 'error');
          }
        );
      } else {
        Swal.fire('Error', 'Geolocalización no soportada.', 'error');
      }
    };

    // Obtener dirección desde coordenadas
    const obtenerDireccion = async (lat, lng) => {
      const url = `https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lng}`;
      try {
        const response = await fetch(url);
        const data = await response.json();
        if (data && data.address) {
          const { road, city, country } = data.address;
          ubicacionUsuario.value.direccion = `${road || ''}, ${city || ''}, ${country || ''}`.trim();
        } else {
          Swal.fire('Error', 'No se pudo obtener la dirección.', 'error');
        }
      } catch (error) {
        console.error("Error al intentar obtener la dirección:", error);
        Swal.fire('Error', 'Ocurrió un error al intentar obtener la dirección.', 'error');
      }
    };

    // Filtrar partidos por deporte y cercanía
    const partidosFiltrados = computed(() => {
      return partidos.value.filter((partido) => {
        const cumpleDeporte = filtroDeporte.value ? partido.deporte === filtroDeporte.value : true;
        const cumpleCercania = filtroCercania.value
          ? calcularDistancia(ubicacionUsuario.value, partido.ubicacion) <= 10
          : true;
        return cumpleDeporte && cumpleCercania;
      });
    });

    // Calcular la distancia entre dos ubicaciones
    const calcularDistancia = (ubicacion1, ubicacion2) => {
      if (!ubicacion1 || !ubicacion2) return Infinity;
      const R = 6371; // Radio de la Tierra en km
      const dLat = (ubicacion2.lat - ubicacion1.lat) * Math.PI / 180;
      const dLng = (ubicacion2.lng - ubicacion1.lng) * Math.PI / 180;
      const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
        Math.cos(ubicacion1.lat * Math.PI / 180) * Math.cos(ubicacion2.lat * Math.PI / 180) *
        Math.sin(dLng / 2) * Math.sin(dLng / 2);
      return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    };

    // Formatear fecha y hora en formato legible
    const formatoFechaHora = (fechaHora) => {
      const fecha = new Date(fechaHora);
      return fecha.toLocaleString("es-ES", { dateStyle: "short", timeStyle: "short" });
    };

    // Abreviar dirección para mostrarla de forma más legible
    const abreviarDireccion = (direccion) => direccion?.split(',').slice(0, 2).join(', ') || "Dirección desconocida";

    // Unirse a un partido
    const unirseAPartido = async (partido) => {
      try {
        if (partido.lugaresDisponibles > 0) {
          const partidoRef = doc(db, "partidos", partido.id);
          await updateDoc(partidoRef, { lugaresDisponibles: partido.lugaresDisponibles - 1 });
          partidoUnido.value = partido;
          modalVisible.value = true;
          Swal.fire('¡Éxito!', `Te has unido al partido de ${partido.nombre}.`, 'success');
        } else {
          Swal.fire('Lo sentimos', 'El partido no tiene lugares disponibles.', 'warning');
        }
      } catch (error) {
        console.error("Error al unirse al partido:", error);
        Swal.fire('Error', 'Ocurrió un error al unirte al partido.', 'error');
      }
    };

    const modalPagoVisible = ref(false); // Controla la visibilidad del modal de pago
    const formPago = ref({ nombreTitular: '', numeroTarjeta: '', fechaExpiracion: '', cvv: '' });

    const abrirModalPago = () => {
      modalVisible.value = false; // Cierra el modal de confirmación
      modalPagoVisible.value = true; // Abre el modal de pago
    };

    const cerrarModalPago = () => {
      modalPagoVisible.value = false;
    };

    const procesarPago = () => {
      // Simulación del proceso de pago
      Swal.fire('Pago exitoso', `El pago para el partido "${partidoUnido.value.nombre}" se ha realizado.`, 'success');
      cerrarModalPago();
    };

    // Cerrar el modal
    const cerrarModal = () => {
      modalVisible.value = false;
      partidoUnido.value = null;
    };

    onMounted(() => {
      cargarPartidos();
    });

    return {
      partidosFiltrados,
      obtenerUbicacionUsuario,
      ubicacionUsuario,
      filtroDeporte,
      filtroCercania,
      unirseAPartido,
      formatoFechaHora,
      abreviarDireccion,
      modalVisible,
      partidoUnido,
      modalVisible,
      modalPagoVisible,
      formPago,
      abrirModalPago,
      cerrarModalPago,
      procesarPago,
      cerrarModal,
      cargando
    };
  },
};
</script>
<style scoped>
button {
  cursor: pointer;
}

button:hover {
  transform: scale(1.02);
}

/* Ajustes para dispositivos móviles */
@media (max-width: 768px) {
  h2 {
    font-size: 1.5rem;
  }

  .text-lg {
    font-size: 0.9rem;
  }

  .text-xl {
    font-size: 1rem;
  }

  .text-gray-600,
  .text-gray-500 {
    font-size: 0.875rem;
  }

  button {
    font-size: 0.875rem;
    padding: 0.5rem 1rem;
  }

  .w-full {
    width: 100%;
  }

  .mb-4 {
    margin-bottom: 1rem;
  }

  .mt-4 {
    margin-top: 1rem;
  }

  .mt-6 {
    margin-top: 1.5rem;
  }

  .space-y-4 {
    gap: 1rem;
  }
}
</style>
