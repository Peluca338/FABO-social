<template>
  <div class="chat-publico h-full">
    <h3 class="chat-title">Chat Público</h3>

    <Loader v-if="cargando" />

    <div v-else class="mensajes-conversacion">
      <div v-for="mensaje in mensajesPublicos" :key="mensaje.id"
        :class="['mensaje', mensaje.userId === user.uid ? 'mensaje-propio' : 'mensaje-otro']">
        <p class="mensaje-contenido">
          <strong>{{ mensaje.usuario }}:</strong> {{ mensaje.texto }}
        </p>
        <span class="mensaje-fecha">{{ formatTimestamp(mensaje.timestamp) }}</span>
      </div>
    </div>

    <div class="input-container">
      <input v-model="mensajeTexto" placeholder="Escribe un mensaje" class="input-message" />
      <button @click="enviarMensaje"
        class="border border-cyan-400 text-white bg-cyan-500 rounded p-2 hover:bg-transparent hover:text-black transition">Enviar</button>
    </div>
  </div>
</template>

<script>
import { getFirestore, collection, addDoc, onSnapshot } from 'firebase/firestore';
import { getAuth } from 'firebase/auth';
import Loader from './Loader.vue';

export default {
  components: {
    Loader,
  },
  data() {
    return {
      mensajesPublicos: [],
      mensajeTexto: '',
      user: null,
      cargando: true,
    };
  },
  created() {
    this.user = getAuth().currentUser;
    this.obtenerMensajesPublicos();
  },
  methods: {
    obtenerMensajesPublicos() {
      const db = getFirestore();
      const chatPublicoRef = collection(db, 'chatPublico');
      onSnapshot(chatPublicoRef, (snapshot) => {
        this.mensajesPublicos = snapshot.docs.map(doc => ({
          id: doc.id,
          ...doc.data(),
        })).sort((a, b) => a.timestamp - b.timestamp);

        this.cargando = false;
      });
    },
    async enviarMensaje() {
      if (this.mensajeTexto.trim()) {
        const db = getFirestore();
        const nuevoMensaje = {
          usuario: this.user.displayName || 'Anónimo',
          texto: this.mensajeTexto,
          timestamp: Date.now(),
          userId: this.user.uid,
        };

        try {
          await addDoc(collection(db, 'chatPublico'), nuevoMensaje);
          this.mensajeTexto = '';
        } catch (error) {
          console.error("Error al enviar el mensaje:", error);
        }
      }
    },
    formatTimestamp(timestamp) {
      const date = new Date(timestamp);
      const optionsDate = { year: 'numeric', month: 'numeric', day: 'numeric' };
      const optionsTime = { hour: '2-digit', minute: '2-digit', hour12: false };
      const formattedDate = date.toLocaleDateString(undefined, optionsDate);
      const formattedTime = date.toLocaleTimeString(undefined, optionsTime);
      return `${formattedDate} ${formattedTime}`;
    }
  },
};
</script>

<style scoped>
.chat-publico {
  padding: 1rem;
  background-color: #f5f5f5;
  border-radius: 8px;
  max-width: 800px;
  margin: 0 auto;
  height: 100%;
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
  max-height: 500px;
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
  flex: 1;
  margin-right: 0.5rem;
}

/* Media Query para dispositivos móviles */
@media (max-width: 768px) {
  .chat-title {
    font-size: 1.2rem;
  }

  .mensaje {
    padding: 0.5rem;
  }

  .mensaje-fecha {
    font-size: 0.65rem;
  }

  .input-message {
    padding: 0.4rem;
  }

  button {
    padding: 0.5rem 1rem;
    font-size: 0.875rem;
  }
}

@media (max-width: 480px) {
  .chat-title {
    font-size: 1.5rem;
  }

  .mensaje {
    padding: 0.6rem;
  }

  .input-message {
    padding: 0.35rem;
  }

  button {
    padding: 0.5rem 0.8rem;
    font-size: 1rem;
  }
}
</style>
