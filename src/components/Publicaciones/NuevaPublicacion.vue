<template>
  <div class="container mx-auto p-4">
    <h2 class="text-2xl font-bold mb-4 ">Crear Nueva Publicación</h2>
    <form @submit.prevent="crearPublicacion" class="bg-white p-6 rounded-lg shadow">
      <!-- Campo de título -->
      <div class="mb-4">
        <label for="titulo" class="block text-gray-700 font-semibold mb-2">Título:</label>
        <input
          type="text"
          v-model="titulo"
          required
          class="border rounded-lg w-full p-2"
        />
      </div>

      <!-- Campo de contenido -->
      <div class="mb-4">
        <label for="contenido" class="block text-gray-700 font-semibold mb-2">Contenido:</label>
        <textarea
          v-model="contenido"
          required
          class="border rounded-lg w-full p-2"
          rows="5"
        ></textarea>
      </div>

      <!-- Campo de imagen -->
      <div class="mb-4">
        <label for="imagen" class="block text-gray-700 font-semibold mb-2">Imagen:</label>
        <input type="file" @change="procesarImagen" accept="image/*" class="border rounded-lg w-full p-2" />
      </div>

      <!-- Botones de acción -->
      <div class="flex justify-center">
        <button
          type="submit"
          class="border border-cyan-400 text-white bg-cyan-500 rounded p-2 hover:bg-transparent hover:text-black transition px-4 py-2  mr-2"
        >
          Crear Publicación
        </button>
        <button
          @click="cancelarCreacion"
          class="border border-amber-400 text-white bg-amber-500 rounded p-2 hover:bg-transparent hover:text-black transition px-4 py-2 "
          type="button"
        >
          Cancelar
        </button>
      </div>
    </form>
  </div>
</template>

<script>
import { getFirestore, collection, addDoc } from "firebase/firestore";
import { getAuth } from "firebase/auth";
import { getStorage, ref, uploadBytes, getDownloadURL } from "firebase/storage";
import Swal from 'sweetalert2';

export default {
  data() {
    return {
      titulo: "",
      contenido: "",
      imagen: null, // Variable para almacenar la imagen seleccionada
    };
  },
  methods: {
    // Método para procesar la imagen seleccionada
    procesarImagen(event) {
      this.imagen = event.target.files[0]; // Guarda la imagen seleccionada
    },
    
    async crearPublicacion() {
      const db = getFirestore();
      const auth = getAuth();
      const storage = getStorage();
      const user = auth.currentUser;

      if (user) {
        try {
          let imagenURL = ""; 
          
          // Si el usuario sube una imagen, la subimos a Firebase Storage
          if (this.imagen) {
            const userId = user.uid; 
            const storageRef = ref(storage, `publicaciones/${userId}/${this.imagen.name}`); 
            const snapshot = await uploadBytes(storageRef, this.imagen);
            imagenURL = await getDownloadURL(snapshot.ref); // Obtén la URL de la imagen subida
          }

          // Guarda la publicación en Firestore con la URL de la imagen
          await addDoc(collection(db, "publicaciones"), {
            titulo: this.titulo,
            contenido: this.contenido,
            imagen: imagenURL, 
            userId: user.uid,
            username: user.displayName || "Usuario Anónimo",
            email: user.email,
            createdAt: new Date()
          });

          // Restablecer los campos del formulario
          this.titulo = "";
          this.contenido = "";
          this.imagen = null;

          Swal.fire({
            icon: 'success',
            title: 'Publicación creada',
            text: '¡Tu publicación se ha creado exitosamente!',
          });

          // Redirigir a la lista de publicaciones
          this.$router.push({ name: 'ListaPublicaciones' });
        } catch (error) {
          console.error("Error al crear la publicación: ", error);

          Swal.fire({
            icon: 'error',
            title: 'Error',
            text: 'Hubo un problema al crear la publicación. Por favor, inténtalo de nuevo.',
          });
        }
      } else {
        Swal.fire({
          icon: 'warning',
          title: 'No autenticado',
          text: 'Debes iniciar sesión para crear una publicación.',
        });
      }
    },

    cancelarCreacion() {
      this.$router.push({ name: 'ListaPublicaciones' }); // Redirige a la lista de publicaciones
    }
  }
};
</script>
<style scoped>
.container {
  width: 100%; 
}

h2 {
  font-size: 1.5rem;
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

.flex {
  justify-content: center; 
}

button {
  padding: 0.50rem 1rem; 
  font-size: 1rem; 
}

/* Media Query para pantallas móviles */
@media (max-width: 768px) {
  h2 {
    font-size: 1.7rem; 
  }

  input, textarea {
    padding: 0.90rem; 
  }

  button {
    padding: 0.7rem 1rem; 
    font-size: 0.950rem; 
  }
}
/* Media Query para pantallas muy pequeñas */
@media (max-width: 480px) {
  h2 {
    font-size: 1.3rem; 
  }

  input, textarea {
    padding: 0.6rem; 
  }

  button {
    padding: 0.6rem 0.8rem; 
    font-size: 0.80rem; 
  }
}
</style>
