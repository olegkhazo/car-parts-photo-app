<template>
  <q-page class="flex flex-center">
    <div v-if="!cameraActive">
      <q-btn label="Begin" @click="startCamera" />
    </div>

    <div v-else class="camera-view">
      <q-btn
        label="DONE"
        flat
        @click="stopCamera"
        style="
          position: absolute;
          top: 10px;
          right: 10px;
          background-color: #dcdad2a6;
          z-index: 10;
          opacity: 0.5;
        "
      />
      <q-btn
        label="Switch Camera"
        flat
        @click="switchCamera"
        style="
          position: absolute;
          top: 10px;
          left: 10px;
          background-color: #dcdad2a6;
          z-index: 10;
          opacity: 0.5;
        "
      />
      <video ref="videoElement" autoplay playsinline></video>

      <q-btn
        icon="photo_camera"
        flat
        @click="takePhoto"
        style="
          position: absolute;
          bottom: 10px;
          left: 50%;
          transform: translateX(-50%);
          background: #dcdad2a6;
          padding: 5px 7px;
          border-radius: 50%;
          opacity: 0.4;
        "
      />

      <canvas ref="canvasElement" style="display: none"></canvas>
    </div>

    <div v-if="photoCollection.length > 0" class="photoCollection-gallery">
      <div
        v-for="(photo, index) in photoCollection"
        :key="index"
        class="photo-item"
      >
        <img :src="photo" />
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, onBeforeUnmount } from "vue";

const videoElement = ref(null);
const canvasElement = ref(null);
const cameraActive = ref(false);
const photoCollection = ref([]);
const facingMode = ref("environment");

const startCamera = async () => {
  cameraActive.value = true;
  try {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: facingMode.value },
    });
    videoElement.value.srcObject = stream;
  } catch (error) {
    console.error("Error accessing the camera:", error);
  }
};

const stopCamera = () => {
  const stream = videoElement.value.srcObject;
  const tracks = stream.getTracks();

  tracks.forEach((track) => track.stop());
  cameraActive.value = false;
};

const switchCamera = async () => {
  // Camera switcher
  facingMode.value = facingMode.value === "user" ? "environment" : "user";

  stopCamera();

  await startCamera();
};

const takePhoto = () => {
  const context = canvasElement.value.getContext("2d");

  canvasElement.value.width = videoElement.value.videoWidth;
  canvasElement.value.height = videoElement.value.videoHeight;
  context.drawImage(videoElement.value, 0, 0);

  const photoData = canvasElement.value.toDataURL("image/png");
  photoCollection.value.push(photoData);

  setTimeout(() => {}, 5000);
};

onBeforeUnmount(() => {
  if (cameraActive.value) {
    stopCamera();
  }
});
</script>

<style scoped>
.camera-view {
  position: relative;
  width: 100%;
  height: 100%;
  background-color: black;
}

video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.photoCollection-gallery {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.photo-item {
  margin: 10px;
}

img {
  max-width: 100px;
  max-height: 100px;
  object-fit: cover;
}
</style>
