<template>
  <div class="max-w-2xl mx-auto p-6 bg-white shadow-md rounded-lg">
    <h2 class="text-3xl sm:text-2xl text-cyan-500 font-bold mb-6">Crear nuevo partido</h2>

    <Loader v-if="cargando" />

    <form v-else @submit.prevent="crearPartido" class="space-y-6">

      <!-- Imagen del Partido -->
      <div>
        <label for="img" class="block text-xl sm:text-lg font-semibold text-gray-700">Imagen del partido</label>
        <input
          class="w-full border border-gray-300 rounded-md p-2 mt-2 focus:outline-none focus:ring-2 focus:ring-blue-500 cursor-pointer sm:p-1"
          type="file" @change="onFileChange" id="img" required />
        <p v-if="imgFile" class="text-gray-600 text-sm mt-2 sm:text-xs">Archivo seleccionado: {{ imgFile.name }}</p>
      </div>

      <!-- Nombre del Partido -->
      <div>
        <label for="name" class="block text-xl sm:text-lg font-semibold text-gray-700">Nombre del partido</label>
        <input v-model="partido.nombre"
          class="w-full border border-gray-300 rounded-md p-2 mt-2 focus:outline-none focus:ring-2 focus:ring-blue-500 sm:p-1"
          type="text" placeholder="Nombre del Partido" required>
      </div>

      <!-- Deporte -->
      <div>
        <label for="deporte" class="block text-xl sm:text-lg font-semibold text-gray-700">Deporte</label>
        <select v-model="partido.deporte"
          class="w-full border border-gray-300 rounded-md p-2 mt-2 focus:outline-none focus:ring-2 focus:ring-blue-500 sm:p-1"
          required>
          <option value="futbol">Fútbol</option>
          <option value="basquet">Básquet</option>
          <option value="tenis">Tenis</option>
        </select>
      </div>

      <!-- Precio del Partido -->
      <div>
        <label for="price" class="block text-xl sm:text-lg font-semibold text-gray-700">Precio del partido</label>
        <input v-model="partido.precio"
          class="w-full border border-gray-300 rounded-md p-2 mt-2 focus:outline-none focus:ring-2 focus:ring-blue-500 sm:p-1"
          type="number" placeholder="Precio del Partido" required min="1">
      </div>

      <!-- Total de Lugares -->
      <div>
        <label for="lugares" class="block text-xl sm:text-lg font-semibold text-gray-700">Total de lugares</label>
        <input v-model="partido.lugaresTotales"
          class="w-full border border-gray-300 rounded-md p-2 mt-2 focus:outline-none focus:ring-2 focus:ring-blue-500 sm:p-1"
          type="number" placeholder="Cantidad de lugares disponibles" required min="1">
      </div>

      <!-- Fecha y Hora del Partido -->
      <div>
        <label for="fecha" class="block text-xl sm:text-lg font-semibold text-gray-700">Fecha y hora del partido</label>
        <input v-model="partido.fechaHora"
          class="w-full border border-gray-300 rounded-md p-2 mt-2 focus:outline-none focus:ring-2 focus:ring-blue-500 sm:p-1"
          type="datetime-local" required>
      </div>

      <!-- Ubicación -->
      <div>
        <label for="ubicacionTexto" class="block text-xl sm:text-lg font-semibold text-gray-700">Buscar
          dirección</label>
        <input v-model="ubicacionTexto" id="ubicacionTexto"
          class="w-full border border-gray-300 rounded-md p-2 mt-2 focus:outline-none focus:ring-2 focus:ring-blue-500 sm:p-1"
          type="text" placeholder="Ingresa una dirección" />
        <button @click.prevent="obtenerCoordenadas"
          class="mt-2 border border-blue-500 text-white bg-blue-500 rounded p-2 hover:bg-transparent hover:text-black transition">
          Buscar Ubicación
        </button>
      </div>
      <div>
        <label class="block text-xl sm:text-lg font-semibold text-gray-700">Selecciona la ubicación</label>
        <div id="map" style="height: 300px; width: 100%; border-radius: 8px;" class="mt-2"></div>
        <p class="text-gray-500 text-sm mt-2 sm:text-xs">Ubicación seleccionada: {{ partido.ubicacion.lat }}, {{
          partido.ubicacion.lng }}</p>
      </div>

      <!-- Descripción del Partido -->
      <div>
        <label for="description" class="block text-xl sm:text-lg font-semibold text-gray-700">Descripción del
          partido</label>
        <textarea v-model="partido.descripcion"
          class="w-full border border-gray-300 rounded-md p-2 mt-2 focus:outline-none focus:ring-2 focus:ring-blue-500 sm:p-1"
          rows="3" placeholder="Descripción del Partido" required></textarea>
      </div>

      <!-- Organizado por -->
      <div v-if="organizador">
        <p class="text-xl sm:text-lg font-semibold text-gray-700">Organizado por: {{ organizador }}</p>
      </div>

      <!-- Botón para enviar -->
      <button type="submit"
        class="border border-cyan-400 text-white bg-cyan-500 rounded p-3 hover:bg-transparent hover:text-black transition px-4 py-2 sm:px-4 sm:py-2 sm:text-base">
        Agregar partido
      </button>
    </form>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue';
import { getFirestore, collection, addDoc } from "firebase/firestore";
import { getStorage, ref as storageRef, uploadBytes, getDownloadURL } from "firebase/storage";
import { getAuth } from "firebase/auth";
import { useRouter } from 'vue-router';
import Swal from 'sweetalert2';
import Loader from './Loader.vue';
import L from 'leaflet';

export default {
  components: {
    Loader,
  },
  setup() {
    const cargando = ref();
    const partido = ref({
      nombre: '',
      precio: 0,
      deporte: '',
      descripcion: '',
      ubicacion: { lat: -34.6083, lng: -58.3712 },
      lugaresTotales: 0,
      lugaresDisponibles: 0,
      fechaHora: '',
    });
    const ubicacionTexto = ref('');
    const imgFile = ref(null);
    const direccion = ref('');
    const organizador = ref(''); // Organizador del partido
    const router = useRouter();

    const auth = getAuth();
    const user = auth.currentUser;

    if (user) {
      organizador.value = user.displayName || user.email;
    }

    onMounted(() => {
      window.map = L.map('map').setView([partido.value.ubicacion.lat, partido.value.ubicacion.lng], 13);
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors'
      }).addTo(window.map);

      window.marker = L.marker([partido.value.ubicacion.lat, partido.value.ubicacion.lng], {
        draggable: true
      }).addTo(window.map);

      window.marker.on('moveend', (e) => {
        const { lat, lng } = e.target.getLatLng();
        partido.value.ubicacion = { lat, lng };
      });
    });

    const crearPartido = async () => {
      if (!user) {
        Swal.fire({
          icon: 'error',
          title: 'Error',
          text: 'Debes estar autenticado para crear un partido.'
        });
        return;
      }

      if (!partido.value.nombre || !partido.value.precio || !partido.value.deporte || !partido.value.descripcion || !partido.value.ubicacion || !partido.value.lugaresTotales || !partido.value.fechaHora) {
        Swal.fire({
          icon: 'warning',
          title: 'Campos incompletos',
          text: 'Por favor, completa todos los campos.'
        });
        return;
      }

      cargando.value = true; // Activar el loader
      try {
        const db = getFirestore();
        const storage = getStorage();
        let imageUrl = '';

        if (imgFile.value) {
          const imgRef = storageRef(storage, `partidos/${user.uid}/${imgFile.value.name}`);
          await uploadBytes(imgRef, imgFile.value);
          imageUrl = await getDownloadURL(imgRef);
        }

        partido.value.lugaresDisponibles = partido.value.lugaresTotales;

        await addDoc(collection(db, "partidos"), {
          ...partido.value,
          imagen: imageUrl,
          fechaCreacion: new Date(),
          direccion: direccion.value,
          userId: user.uid,
          organizadoPor: organizador.value,
        });

        Swal.fire({
          icon: 'success',
          title: 'Éxito',
          text: 'Partido creado exitosamente'
        });
        router.push('/partidos');
      } catch (error) {
        console.error("Error al crear el partido:", error);
        Swal.fire({
          icon: 'error',
          title: 'Error',
          text: 'Ocurrió un error al crear el partido.'
        });
      } finally {
        cargando.value = false;
      }
    };


    const onFileChange = (e) => {
      imgFile.value = e.target.files[0];
    };

    const obtenerCoordenadas = async () => {
      const res = await fetch(`https://nominatim.openstreetmap.org/search?q=${ubicacionTexto.value}&format=json&addressdetails=1`);
      const data = await res.json();
      if (data.length > 0) {
        partido.value.ubicacion.lat = parseFloat(data[0].lat);
        partido.value.ubicacion.lng = parseFloat(data[0].lon);
        direccion.value = data[0].display_name;
        window.marker.setLatLng([partido.value.ubicacion.lat, partido.value.ubicacion.lng]);
        window.map.setView([partido.value.ubicacion.lat, partido.value.ubicacion.lng], 13);
      }
    };

    return {
      partido,
      ubicacionTexto,
      imgFile,
      organizador,
      crearPartido,
      onFileChange,
      obtenerCoordenadas,
      cargando
    };
  }
};
</script>

<style scoped>
/* Estilos generales */
input[type="file"],
button {
  font-size: 1rem;
  /* Tamaño de fuente predeterminado */
}

button:hover {
  background-color: transparent;
  color: black;
}

/* Ajustes para pantallas más pequeñas */
@media (max-width: 640px) {
  h2 {
    font-size: 1.4rem;
    /* Ajustar título en móvil */
  }

  .text-xl {
    font-size: 1rem;
    /* Reducir tamaño de texto en campos */
  }

  .text-lg {
    font-size: 0.875rem;
    /* Reducir tamaño en campos de selección */
  }

  .sm:text-lg {
    font-size: 0.875rem;
    /* Reducir tamaño en campos pequeños */
  }

  .sm:text-base {
    font-size: 0.75rem;
    /* Texto más pequeño en móvil */
  }

  .sm:text-xs {
    font-size: 0.625rem;
    /* Texto extra pequeño */
  }

  .sm:p-1,
  .sm:px-3,
  .sm:py-2 {
    padding: 0.4rem;
    /* Ajustar tamaño de los botones */
  }
}
</style>
