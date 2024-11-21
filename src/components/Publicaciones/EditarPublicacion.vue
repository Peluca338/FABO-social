<template>
  <div class="container mx-auto p-4">
    <h2 class="text-2xl font-bold mb-4">Editar Publicación</h2>
    <form @submit.prevent="actualizarPublicacion" class="bg-white p-6 rounded-lg shadow-md">
      <div class="mb-4">
        <label for="titulo" class="block text-gray-700 font-semibold mb-2">Título:</label>
        <input type="text" v-model="titulo" required class="border rounded-lg w-full p-2" />
      </div>
      <div class="mb-4">
        <label for="contenido" class="block text-gray-700 font-semibold mb-2">Contenido:</label>
        <textarea v-model="contenido" required class="border rounded-lg w-full p-2"></textarea>
      </div>
      <div class="flex justify-center space-x-4">
        <button type="submit" class="border border-cyan-400 text-white bg-cyan-500 rounded p-2 hover:bg-transparent hover:text-black transition px-4 py-2">
          Actualizar
        </button>
        <router-link to="/" class="border border-amber-400 text-white bg-amber-500 rounded p-2 hover:bg-transparent hover:text-black transition px-4 py-2">
          Cancelar
        </router-link>
      </div>
    </form>
  </div>
</template>

<script>
import { getFirestore, doc, getDoc, updateDoc } from 'firebase/firestore';

export default {
  data() {
    return {
      titulo: '',
      contenido: '',
      id: this.$route.params.id,
    };
  },
  created() {
    console.log("Cargando publicación con ID:", this.id);
    this.cargarPublicacion(this.id);
  },
  methods: {
    async cargarPublicacion(id) {
      const db = getFirestore();
      const docRef = doc(db, "publicaciones", id);
      const docSnap = await getDoc(docRef);
      if (docSnap.exists()) {
        this.titulo = docSnap.data().titulo;
        this.contenido = docSnap.data().contenido;
      } else {
        alert("No se encontró la publicación.");
      }
    },
    async actualizarPublicacion() {
      const db = getFirestore();
      const docRef = doc(db, "publicaciones", this.id);

      try {
        await updateDoc(docRef, {
          titulo: this.titulo,
          contenido: this.contenido
        });
        alert("Publicación actualizada exitosamente.");
        this.$router.push({ name: 'ListaPublicaciones' });
      } catch (error) {
        alert("Error al actualizar la publicación: " + error.message);
      }
    }
  }
};
</script>

<style scoped>
.container {
  width: 100%;
}
h2 {
  font-size: 1.6rem;
  margin-bottom: 1rem;
  color: #282828;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
  letter-spacing: 0.05em;
}

.bg-white {
  width: 100%;
  border-radius: 0.5rem;
}

button, .router-link {
  padding: 0.50rem 1rem;
  font-size: 1rem;
  border-radius: 0.375rem;
  transition: all 0.3s ease;
}

/* Media Query para pantallas móviles */
@media (max-width: 768px) {
  h2 {
    font-size: 1.5rem;
    text-align: center; 
  }

  input, textarea {
    padding: 0.75rem; 
  }

  button, .router-link {
    padding: 0.5rem 1rem; 
    font-size: 0.875rem; 
  }
}
/* Media Query para pantallas muy pequeñas */
@media (max-width: 480px) {
  h2 {
    font-size: 1.2rem;
    text-align: center; 
  }

  input, textarea {
    padding: 0.5rem; 
  }

  button, .router-link {
    padding: 0.5rem 0.80rem; 
    font-size: 0.75rem; 
  }
}
</style>
