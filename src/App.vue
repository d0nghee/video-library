<template>
  <div>
    <base-btn
      :name="'vue-video-player'"
      @click="currentComponent = 'vue-video-player'"
    ></base-btn>
    <base-btn
      :name="'canvas-video-player'"
      @click="currentComponent = 'canvas-video-player'"
    ></base-btn>

    <component :is="currentComponent"></component>
  </div>
</template>

<script>
import BaseBtn from "./components/BaseBtn.vue";
import VueVideoPlayer from "./components/VueVideoPlayer.vue";
import CanvasVideoPlayer from "./components/CanvasVideoPlayer.vue";

export default {
  components: {
    BaseBtn,
    VueVideoPlayer,
    CanvasVideoPlayer,
  },
  data() {
    return {
      currentComponent: null,
    };
  },
};
</script>

<style scoped></style>
