<template>
  <div class="comentarios mt-4">
    <h4 class="text-lg sm:text-base font-semibold">Comentarios</h4>
    <form @submit.prevent="agregarComentario" class="space-y-2 mb-4">
      <input type="text" v-model="nuevoComentario" placeholder="Escribe un comentario..."
        class="border rounded p-2 w-full text-sm sm:text-base" required />
      <button type="submit"
        class="border border-blue-500 text-white bg-blue-500 rounded p-2 hover:bg-transparent hover:text-black transition text-sm sm:text-base">
        Agregar comentario
      </button>
    </form>
    <ul>
      <li v-for="(comentario, index) in comentarios" :key="comentario.id" class="p-2 border-b last:border-0">
        <strong class="text-sm sm:text-base">{{ comentario.username }}</strong>: {{ comentario.texto }}
        <button v-if="comentario.userId === auth.currentUser?.uid" @click="eliminarComentario(comentario.id)"
          class="text-red-500 ml-2 text-xs sm:text-sm">
          Eliminar
        </button>
      </li>
    </ul>
  </div>
</template>

<script>
import { getFirestore, collection, addDoc, getDocs, query, where, deleteDoc, doc } from "firebase/firestore";
import { getAuth } from "firebase/auth";
import Swal from 'sweetalert2';

export default {
  props: ['publicacionId'],
  data() {
    return {
      nuevoComentario: '',
      comentarios: [],
      auth: getAuth() // Obteniendo la instancia de autenticación
    };
  },
  async mounted() {
    await this.obtenerComentarios();
  },
  methods: {
    async obtenerComentarios() {
      const db = getFirestore();
      const comentariosRef = collection(db, "comentarios");
      const q = query(comentariosRef, where("publicacionId", "==", this.publicacionId));
      const comentariosSnapshot = await getDocs(q);

      // Mapear los comentarios a un arreglo con id, texto, userId y username
      this.comentarios = comentariosSnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
    },
    async agregarComentario() {
      if (this.nuevoComentario.trim() !== '') {
        const db = getFirestore();
        const user = this.auth.currentUser; // Obtener el usuario autenticado

        try {
          const docRef = await addDoc(collection(db, "comentarios"), {
            publicacionId: this.publicacionId,
            texto: this.nuevoComentario.trim(),
            username: user ? user.displayName || "Usuario Anónimo" : "Usuario Anónimo",
            userId: user ? user.uid : null, // Agregar el userId 
            createdAt: new Date()
          });

          // Agregar el comentario al arreglo local para mostrar inmediatamente
          this.comentarios.push({
            id: docRef.id,
            texto: this.nuevoComentario.trim(),
            username: user ? user.displayName || "Usuario Anónimo" : "Usuario Anónimo",
            userId: user ? user.uid : null
          });
          this.nuevoComentario = ''; // Limpiar el campo
        } catch (error) {
          console.error("Error al agregar el comentario: ", error);
        }
      } else {
        alert("El comentario no puede estar vacío.");
      }
    },
    async eliminarComentario(id) {
      Swal.fire({
        title: '¿Estás seguro?',
        text: "No podrás revertir esta acción.",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#d33',
        cancelButtonColor: '#3085d6',
        confirmButtonText: 'Sí, eliminar',
        cancelButtonText: 'Cancelar'
      }).then(async (result) => {
        if (result.isConfirmed) {
          const db = getFirestore();
          try {
            await deleteDoc(doc(db, "comentarios", id)); // Eliminar de Firestore
            this.comentarios = this.comentarios.filter(comentario => comentario.id !== id);
            Swal.fire(
              'Eliminado!',
              'El comentario ha sido eliminado.',
              'success'
            );
          } catch (error) {
            console.error("Error al eliminar el comentario: ", error);
            Swal.fire(
              'Error',
              'Hubo un problema al eliminar el comentario. Inténtalo de nuevo.',
              'error'
            );
          }
        }
      });
    }
  }
};
</script>

<style scoped>
.comentarios {
  background-color: #f9fafb;
  border-radius: 8px;
  padding: 16px;
}

button {
  transition: background-color 0.3s ease;
}

ul {
  list-style-type: none;
  padding: 0;
}

.button-comentario {
  width: auto;
  padding: 0.5rem 1rem;
  display: block;
}

@media (max-width: 640px) {
  .text-sm {
    font-size: 0.875rem; 
  }
  .sm\:text-base {
    font-size: 1rem; 
  }
  .p-2 {
    padding: 0.5rem; 
  }
  .ml-2 {
    margin-left: 0.5rem; 
  }
}
</style>
