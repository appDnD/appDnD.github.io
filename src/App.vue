<script setup>
import { onMounted, watch } from 'vue';
import { storeToRefs } from 'pinia';
import AppHeader from './components/AppHeader.vue';
import Swal from 'sweetalert2';
import { useAccountStore } from './stores/useAccountStore';
import { useCharacterStore } from './stores/useCharacterStore';

const accountStore = useAccountStore();
const { isLoading } = storeToRefs(accountStore);

onMounted(async () => {
  // 1. Cargar datos locales de forma síncrona
  accountStore.loadInitialData();

  // 2. Comprobar si es un usuario nuevo (no tiene ID asignado)
  if (!accountStore.accountData.accountId) {
    const { value: username } = await Swal.fire({
      title: '¡Bienvenido a DyD!',
      html: '<p>Por favor, elige un <b>nombre de usuario</b> único. Este será tu ID para sincronizar y recuperar datos en la nube.</p><p style="font-size: 0.85em; color: #b9bbbe;">Si ya tienes una cuenta en la nube, introduce aquí mismo tu nombre de usuario para recuperarla.</p>',
      input: 'text',
      inputPlaceholder: 'Ej: aventurero-123',
      allowOutsideClick: false,
      allowEscapeKey: false,
      confirmButtonColor: '#3498db',
      confirmButtonText: 'Entrar',
      customClass: { container: 'high-z-index' },
      inputValidator: (value) => {
        if (!value) {
          return '¡Necesitas un nombre de usuario!';
        }
        if (value.length < 3) {
          return 'El nombre debe tener al menos 3 caracteres.';
        }
      }
    });
    
    if (username) {
      const formattedId = username.trim().toLowerCase().replace(/\s+/g, '-');
      
      Swal.fire({
        title: 'Conectando con la nube...',
        text: 'Comprobando si tu usuario ya existe...',
        allowOutsideClick: false,
        didOpen: () => { Swal.showLoading(); },
        customClass: { container: 'high-z-index' }
      });

      // Intentamos cargar de Google a ver si ya existía ese nombre
      const { success } = await accountStore.loadFromGoogle(formattedId);
      
      if (success) {
        // Si success es true, loadFromGoogle hace un reload y recarga la página con los datos.
        return;
      } else {
        // Si no existe en la nube o hay error de conexión, asumimos que es una cuenta nueva y la creamos.
        accountStore.accountData.accountId = formattedId;
        accountStore.saveDataToLocalStorage();
        
        Swal.fire({
          icon: 'success',
          title: '¡Cuenta creada!',
          text: `Tu ID de usuario es: ${formattedId}`,
          timer: 3000,
          showConfirmButton: false,
          customClass: { container: 'high-z-index' }
        });
      }
    }
  }

  // 3. Una vez que el ID está asegurado o cargado, inicializamos los demás stores
  const characterStore = useCharacterStore();
  characterStore.loadAllCharacterData();
});
</script>

<template>
  <div class="app-container">
    <AppHeader />
    
    <main class="main-content">
      <RouterView />
    </main>
  </div>
</template>

<style>
/* Estilos globales para la aplicación */
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  overflow-x: hidden; /* Prevent horizontal scroll on mobile */
}

.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #0f0f23 0%, #1a1a2e 50%, #16213e 100%);
}

.main-content {
  padding: 0;
  min-height: calc(100vh - 60px); /* Updated to match new header height */
}

/* Scrollbar personalizado */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: #1a1a2e;
}

::-webkit-scrollbar-thumb {
  background: #f39c12;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #e67e22;
}

/* Animaciones globales */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-in {
  animation: fadeIn 0.6s ease-out;
}

/* Mejoras para Bootstrap */
.btn {
  transition: all 0.3s ease;
}

.card {
  transition: all 0.3s ease;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0,0,0,0.2);
}

/* Mobile-specific improvements */
@media (max-width: 768px) {
  .main-content {
    min-height: calc(100vh - 65px);
  }
  
  /* Prevent text selection on mobile for better UX */
  .btn, .nav-link, .action-btn {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }
  
  /* Improve touch targets */
  button, .btn, .nav-link {
    min-height: 44px; /* Apple's recommended minimum touch target size */
  }
}

@media (max-width: 480px) {
  .main-content {
    min-height: calc(100vh - 60px);
  }
}

/* Touch device optimizations */
@media (hover: none) and (pointer: coarse) {
  .card:hover {
    transform: none;
  }
  
  /* Add active states for touch devices */
  .btn:active,
  .action-btn:active {
    transform: scale(0.98);
  }
}
</style>
