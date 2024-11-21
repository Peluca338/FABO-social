<template>
  <div class="chat-privado">
    <h3 class="chat-title">
      Chat Privado con: {{ destinatarioNombre }}
    </h3>

    <!-- Mensajes de la conversación seleccionada -->
    <div v-if="conversacionActual.length" class="mensajes-conversacion">
      <div
        v-for="mensaje in conversacionActual"
        :key="mensaje.id"
        :class="['mensaje', mensaje.usuario === usuarioActualId ? 'mensaje-propio' : 'mensaje-otro']"
      >
        <p class="mensaje-contenido">
          <strong>{{ mensaje.usuarioNombre }}:</strong> {{ mensaje.texto }}
        </p>
        <span class="mensaje-fecha">{{ formatFecha(mensaje.timestamp) }}</span>
      </div>
    </div>

    <!-- Input para enviar mensajes -->
    <div v-if="destinatarioId" class="input-container">
      <input v-model="mensajePrivadoTexto" placeholder="Escribe un mensaje" class="input-message" />
      <button @click="enviarMensajePrivado" class="border border-cyan-400 text-white bg-cyan-500 rounded p-2 hover:bg-transparent hover:text-black transition text-sm sm:text-base">Enviar</button>
    </div>

    <!-- Mensaje de bienvenida si no hay conversaciones -->
    <div v-if="!conversacionActual.length && !destinatarioId" class="no-conversacion">
      <p>Seleccione un usuario para iniciar una conversación.</p>
    </div>
  </div>
</template>

<script>
import { getFirestore, collection, addDoc, onSnapshot } from 'firebase/firestore';
import { getAuth } from 'firebase/auth';

export default {
  data() {
    return {
      conversacionActual: [],
      mensajePrivadoTexto: '',
      destinatarioId: this.$route.params.destinatarioId,
      destinatarioNombre: this.$route.params.destinatarioNombre || 'Usuario Desconocido',
      usuarioActualId: null, // ID del usuario actual para identificar mensajes propios
    };
  },
  created() {
    this.cargarConversacion();
    const auth = getAuth();
    this.usuarioActualId = auth.currentUser.uid; // Guarda el ID del usuario actual
  },
  methods: {
    cargarConversacion() {
      const db = getFirestore();
      onSnapshot(collection(db, 'chatPrivado'), (snapshot) => {
        this.conversacionActual = snapshot.docs
          .map(doc => ({ id: doc.id, ...doc.data() }))
          .filter(mensaje => 
            (mensaje.usuario === this.usuarioActualId && mensaje.destinatario === this.destinatarioId) ||
            (mensaje.destinatario === this.usuarioActualId && mensaje.usuario === this.destinatarioId)
          );

        // Obtener el nombre del destinatario desde la conversación actual
        const otroUsuario = this.conversacionActual.find(mensaje => mensaje.usuario !== this.usuarioActualId);
        if (otroUsuario) {
          this.destinatarioNombre = otroUsuario.usuarioNombre; // Actualiza el nombre del destinatario
        }
      });
    },
    async enviarMensajePrivado() {
      if (this.mensajePrivadoTexto.trim() === '' || !this.destinatarioId) return;
      const db = getFirestore();
      const auth = getAuth();
      const usuario = auth.currentUser;

      try {
        await addDoc(collection(db, 'chatPrivado'), {
          usuario: usuario.uid,
          destinatario: this.destinatarioId,
          texto: this.mensajePrivadoTexto,
          usuarioNombre: usuario.displayName || 'Anónimo',
          timestamp: Date.now(),
        });
        this.mensajePrivadoTexto = '';
      } catch (error) {
        console.error("Error al enviar el mensaje privado: ", error);
      }
    },
    formatFecha(timestamp) {
      const fecha = new Date(timestamp);
      return `${fecha.toLocaleDateString()} ${fecha.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}`;
    },
  },
};
</script>

<style scoped>
.chat-privado {
  padding: 1rem;
  background-color: #f5f5f5;
  border-radius: 8px;
  max-width: 600px;
  margin: 0 auto;
}

.chat-title {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: #282828;
  text-align: center;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.3);
  letter-spacing: 0.05em;
}

.mensajes-conversacion {
  margin: 1rem 0;
  max-height: 400px;
  overflow-y: auto;
  background-color: #ffffff;
  padding: 1rem;
  border-radius: 8px;
}

.mensaje {
  margin-bottom: 1rem;
  padding: 0.75rem;
  border-radius: 8px;
  max-width: 75%;
  position: relative;
  word-wrap: break-word;
}

.mensaje-propio {
  background-color: #cfe9ff;
  align-self: flex-end;
  margin-left: auto;
  text-align: right;
}

.mensaje-otro {
  background-color: #ffe0b2;
  align-self: flex-start;
  margin-right: auto;
  text-align: left;
}

.mensaje-contenido {
  margin: 0;
}

.mensaje-fecha {
  font-size: 0.75rem;
  color: #555;
  margin-top: 0.25rem;
  display: block;
}

.input-container {
  display: flex;
  margin-top: 1rem;
}

.input-message {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  margin-right: 0.5rem;
  flex-grow: 1;
}

.no-conversacion {
  margin-top: 1rem;
  color: #777;
  text-align: center;
}

/* Responsividad */
@media (max-width: 640px) {
  .chat-title {
    font-size: 1.25rem; 
  }

  .mensaje {
    padding: 0.5rem;
    font-size: 0.875rem; 
  }

  .input-message {
    padding: 0.4rem; 
  }

  button {
    padding: 0.5rem 1rem; 
    font-size: 0.875rem;
  }

  .no-conversacion p {
    font-size: 0.875rem; 
  }
}
</style>
