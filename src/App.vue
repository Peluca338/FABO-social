<template>
  <div class="app">
    <!-- Barra de navegación para PC -->
    <nav class="bg-zinc-100 p-4 shadow lg:block hidden">
      <div class="container mx-auto flex items-center justify-between">
        
        <!-- Enlaces a la izquierda (para PC) -->
        <div class="flex space-x-6 lg:flex-none">
          <router-link v-if="!isLoggedIn" to="/bienvenidos" class="text-black hover:text-cyan-400 cursor-pointer hover:scale-105 transition font-medium">Bienvenidos</router-link>
          <router-link to="/" class="text-black hover:text-cyan-400 cursor-pointer hover:scale-105 transition font-medium">Comunidad</router-link>
          <router-link v-if="isLoggedIn" to="/crear-partido" class="text-black hover:text-cyan-400 cursor-pointer hover:scale-105 transition font-medium">Crear Partido</router-link>
          <router-link v-if="isLoggedIn" to="/partidos" class="text-black hover:text-cyan-400 cursor-pointer hover:scale-105 transition font-medium">Partidos</router-link>
        </div>

        <!-- Logo centrado (para PC) -->
        <div class="lg:mx-auto flex justify-center lg:flex-none">
          <router-link to="/">
            <img src="/img/logo.png" alt="FABO Logo" class="w-32 h-auto" />
          </router-link>
        </div>

        <!-- Enlaces a la derecha (para PC) -->
        <div class="flex space-x-6 items-center">
          <router-link v-if="isLoggedIn" to="/perfil" class="flex items-center justify-center w-10 h-10 rounded-full bg-cyan-400 text-white hover:bg-blue-700 transition">
            <i class="fa-solid fa-user fa-lg"></i>
          </router-link>
          <router-link v-if="!isLoggedIn" to="/login" class="text-black hover:text-cyan-400 cursor-pointer hover:scale-105 transition font-medium">Iniciar Sesión</router-link>
          <router-link v-if="!isLoggedIn" to="/registro" class="text-black hover:text-cyan-400 cursor-pointer hover:scale-105 transition font-medium">Registro</router-link>
          <span v-if="isLoggedIn" class="text-cyan-400 font-medium">Hola, {{ userEmail }}</span>
          <span v-if="isLoggedIn" @click="logout" class="text-black hover:text-cyan-400 cursor-pointer hover:scale-105 transition font-medium">Cerrar Sesión</span>
        </div>
      </div>
    </nav>

    <!-- Menú hamburguesa (tablet y móvil) -->
    <div class="lg:hidden">
      <!-- Barra de navegación con el logo centrado y el menú hamburguesa -->
      <nav class="bg-zinc-100 p-4 shadow">
        <div class="container mx-auto flex items-center justify-between">
          <!-- Botón de menú (hamburguesa) -->
          <button 
            @click="menuAbierto = !menuAbierto" 
            class="text-black lg:hidden focus:outline-none"
          >
            <svg 
              xmlns="http://www.w3.org/2000/svg" 
              class="h-6 w-6" 
              fill="none" 
              viewBox="0 0 24 24" 
              stroke="currentColor"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                d="M4 6h16M4 12h16m-7 6h7" 
              />
            </svg>
          </button>

          <!-- Logo centrado (para tablet y móvil) -->
          <div class="flex justify-center w-full">
            <router-link to="/">
              <img src="/img/logo.png" alt="FABO Logo" class="w-32 h-auto" />
            </router-link>
          </div>
        </div>
      </nav>

      <!-- Menú desplegable (hamburguesa) -->
      <div v-if="menuAbierto" class="mt-4">
        <ul class="flex flex-col items-start space-y-2 pl-4">
          <!-- Enlaces de navegación y sesiones alineados a la izquierda -->
          <li>
            <router-link to="/bienvenidos" class="text-black hover:text-cyan-400 font-medium">Bienvenidos</router-link>
          </li>
          <li>
            <router-link to="/" class="text-black hover:text-cyan-400 font-medium">Comunidad</router-link>
          </li>
          <li v-if="isLoggedIn">
            <router-link to="/crear-partido" class="text-black hover:text-cyan-400 font-medium">Crear Partido</router-link>
          </li>
          <li v-if="isLoggedIn">
            <router-link to="/partidos" class="text-black hover:text-cyan-400 font-medium">Partidos</router-link>
          </li>
          <li v-if="isLoggedIn">
            <router-link to="/perfil" class="text-black hover:text-cyan-400 font-medium">Perfil</router-link>
          </li>
          <li v-if="!isLoggedIn">
            <router-link to="/login" class="text-black hover:text-cyan-400 font-medium">Iniciar Sesión</router-link>
          </li>
          <li v-if="!isLoggedIn">
            <router-link to="/registro" class="text-black hover:text-cyan-400 font-medium">Registro</router-link>
          </li>
          <li v-if="isLoggedIn" @click="logout" class="text-black hover:text-cyan-400 font-medium cursor-pointer">Cerrar Sesión</li>
        </ul>
      </div>
    </div>

    <!-- Barra de navegación inferior para móviles y tabletas -->
<div class="lg:hidden fixed bottom-0 w-full bg-cyan-500 shadow-lg z-10">
  <div class="flex justify-between p-4">
    <router-link to="/" class="text-white flex flex-col items-center">
      <i class="fa-solid fa-house fa-lg"></i>
      <span class="text-xs mt-3">Comunidad</span>
    </router-link>
    <router-link v-if="isLoggedIn" to="/partidos" class="text-white flex flex-col items-center">
      <i class="fa-solid fa-futbol fa-lg"></i>
      <span class="text-xs mt-3">Partidos</span>
    </router-link>
    <router-link v-if="isLoggedIn" to="/perfil" class="text-white flex flex-col items-center">
      <i class="fa-solid fa-user fa-lg"></i>
      <span class="text-xs mt-3">Perfil</span>
    </router-link>
  </div>
</div>



    <!-- Contenido -->
    <main class="container mx-auto mt-4 mb-16">
      <router-view />
    </main>

    <!-- Footer -->
    <footer class="bg-cyan-500 text-white py-4 mt-8">
      <div class="container mx-auto text-center">
        <p>&copy; 2024 FABO - Fútbol, Amistad, Básquet, Otros</p>
      </div>
    </footer>
  </div>
</template>

<script>
import { getAuth, signOut } from "firebase/auth";

export default {
  data() {
    return {
      isLoggedIn: false,
      userEmail: '',
      menuAbierto: false, 
    };
  },
  mounted() {
    const auth = getAuth();
    this.isLoggedIn = !!auth.currentUser;

    // Escucha cambios en el estado de autenticación
    auth.onAuthStateChanged(user => {
      this.isLoggedIn = !!user;
      if (user) {
        this.userEmail = user.email;
      }
    });
  },
  methods: {
    async logout() {
      const auth = getAuth();
      await signOut(auth);
      this.isLoggedIn = false;
      this.$router.push("/login");
    },
  },
};
</script>

<style scoped>
@import "tailwindcss/tailwind.css";
</style>
