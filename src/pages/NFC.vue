<template>
    <div>
      <button @click="startNFC">Iniciar NFC</button>
    </div>
  </template>
  
  <script>
  import { NFC } from '@ionic-native/nfc';
  
  export default {
    methods: {
      startNFC() {
        NFC.enabled().then(() => {
          // El NFC está habilitado, puedes continuar con la lógica para escanear el DNI.
          this.scanDNI();
        }).catch((error) => {
          console.error('NFC no está habilitado', error);
        });
      },
  
      scanDNI() {
        // Implementa la lógica para escanear el DNI aquí utilizando el lector NFC.
        // Puedes utilizar eventos para manejar la lectura del NFC y obtener los datos del DNI.
        // Ejemplo: NFC.addNdefListener(() => { /* Manejar la lectura NFC aquí */ });
      },
    },
  };
  </script>