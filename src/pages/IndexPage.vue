<template>
  <q-page>
    <div style="display: flex; justify-content: center; align-items: center; height: 100vh;">
      <div style="position: relative; width: 3600px; height: 2160px; overflow: hidden;">
        <img src="../assets/dni.png" alt="dni background" style="position: absolute; width: 100%; height: 100%; z-index: -1;">
        <video ref="video" width="3600" height="2160" autoplay style="position: absolute; display: none;"></video>
        <canvas ref="dniFrameCanvas" style="position: absolute; border: 6px solid red; box-sizing: border-box; transform: translate(-50%, -50%);" :style="dniFrameStyle" @click="captureIfSameColor"></canvas>
      </div>
    </div>
  </q-page>
</template>

<script>
export default {
  data() {
    return {
      dniFrameStyle: {
        width: '90%',
        height: '25%',
        position: 'absolute',
        top: '77%',
        left: '50%',
        transform: 'translate(-50%, -50%)',
      },
      captureThreshold: 50, // Umbral de diferencia de color para considerar que los bordes son del mismo color
    };
  },
  methods: {
    takePhoto() {
      const video = this.$refs.video;
      const canvas = this.$refs.dniFrameCanvas;
      const context = canvas.getContext('2d');

      // Obtén las dimensiones de la región de interés (ROI)
      const roiWidth = video.width * parseFloat(this.dniFrameStyle.width) / 100;
      const roiHeight = video.height * parseFloat(this.dniFrameStyle.height) / 100;

      // Configura el lienzo con las dimensiones de la región de interés
      canvas.width = roiWidth;
      canvas.height = roiHeight;

      // Dibuja el fotograma actual del video en la región de interés del lienzo
      context.drawImage(video, -video.width * (parseFloat(this.dniFrameStyle.left) / 100) + roiWidth / 2, -video.height * (parseFloat(this.dniFrameStyle.top) / 100) + roiHeight / 2, video.width, video.height);

     // Aplica el filtro de escala de grises con aumento de brillo
     const imageData = context.getImageData(0, 0, roiWidth, roiHeight);
      const data = imageData.data;
      for (let i = 0; i < data.length; i += 4) {
        const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
        const brightenedAvg = avg * 1.5; // Ajusta este valor para aumentar el brillo
        data[i] = brightenedAvg;
        data[i + 1] = brightenedAvg;
        data[i + 2] = brightenedAvg;
      }
      context.putImageData(imageData, 0, 0);

      // Dibuja el borde en la región de interés
      context.lineWidth = 2;
      context.strokeRect(0, 0, roiWidth, roiHeight);

      return imageData; // Devuelve los datos de imagen para análisis de color
    },
    isSameColor(color1, color2) {
      // Compara si dos colores son iguales dentro del umbral de diferencia
      const colorDiff = Math.abs(color1 - color2);
      return colorDiff <= this.captureThreshold;
    },
    captureIfSameColor() {
      const imageData = this.takePhoto();
      const data = imageData.data;

      // Obtén el color de los píxeles en los bordes de la región de interés
      const topLeftColor = data[0];
      const topRightColor = data[(imageData.width - 1) * 4];
      const bottomLeftColor = data[(imageData.height - 1) * imageData.width * 4];
      const bottomRightColor = data[(imageData.height - 1) * imageData.width * 4 + (imageData.width - 1) * 4];

      // Comprueba si los cuatro bordes tienen el mismo color
      const isSameColorOnAllEdges = this.isSameColor(topLeftColor, topRightColor) &&
        this.isSameColor(topLeftColor, bottomLeftColor) &&
        this.isSameColor(topLeftColor, bottomRightColor);

      if (isSameColorOnAllEdges) {
        // Si los bordes tienen el mismo color, realiza la acción que desees
        console.log('Captura realizada porque los bordes tienen el mismo color.');
        // Imprimir la foto en la consola
          // Obtener la representación base64 de la imagen
        const base64Image = this.$refs.dniFrameCanvas.toDataURL("image/jpeg", 1.0);
        console.log('Imagen capturada:', base64Image);
  
        // Puedes agregar aquí la lógica para guardar la captura, etc.
      } else {
        console.log('Los bordes no tienen el mismo color. Captura no realizada.');
      }
    },
    initCamera() {
      navigator.mediaDevices
        .getUserMedia({ video: true })
        .then((stream) => {
          const video = this.$refs.video;
          video.srcObject = stream;

          // Configura un bucle para tomar fotos continuamente
          const takePhotoLoop = () => {
            this.takePhoto();
            requestAnimationFrame(takePhotoLoop);
          };

          // Inicia el bucle
          takePhotoLoop();
        })
        .catch((error) => {
          console.error('Error al acceder a la cámara:', error);
        });
    },
  },
  mounted() {
    this.initCamera();
  },
};
</script>
