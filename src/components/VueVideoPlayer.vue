<template>
  <div>
    <h3>vue-video-player</h3>
    <video-player
      ref="videoPlayer"
      :options="videoOptions"
      @loadstart="onLoadStart"
      @error="onError"
      @width="500"
    >
      <template v-slot="{ player, state }">
        <div class="custom-player-controls">
          <button @click="state.playing ? player.pause() : player.play()">
            {{ state.playing ? "Pause" : "Play" }}
          </button>
          <button @click="player.muted(!state.muted)">
            {{ state.muted ? "UnMute" : "Mute" }}
          </button>

          <button @click="logVideoTime(player)">현재 시간 출력</button>
          <button @click="nextFrame(player)">다음 프레임</button>
          <button @click="playFrameLevel(player, state)">
            프레임단위 재생
          </button>
        </div>
      </template>
    </video-player>
  </div>
</template>

<script>
import { VideoPlayer } from "vue-video-player";
import "video.js/dist/video-js.css";

export default {
  components: {
    VideoPlayer,
  },
  data() {
    return {
      framePlay: false,
      videoOptions: {
        autoplay: false,
        controls: true,
        sources: [{ src: "/video/tiny.mp4", type: "video/mp4" }],
      },
      fps: 30,
      frameInterval: null,
    };
  },
  methods: {
    onLoadStart() {
      console.log("시작");
    },
    onError() {
      console.log("오류");
    },
    logVideoTime(player) {
      console.log("현재 시간: ", player.currentTime()); // 현재 시간
      console.log("전체 길이: ", player.duration()); // 전체 길이
      console.log("재생 속도: ", player.playbackRate()); // 재생 속도
    },
    nextFrame(player) {
      let currentTimeInSeconds = player.currentTime();

      const nextFrameTime = currentTimeInSeconds + 1 / this.fps;
      player.currentTime(nextFrameTime);
    },
    playFrameLevel(player) {
      player.playbackRate(0);
      player.play();

      if (this.frameInterval) {
        clearTimeout(this.frameInterval);
        this.frameInterval = null;
      }
      const playNextFrame = () => {
        if (player.currentTime() == null) {
          return;
        }
        let currentTimeInSeconds = player.currentTime();
        let nextFrameTime = currentTimeInSeconds + 1 / this.fps;

        if (currentTimeInSeconds >= player.duration()) {
          console.log();
          clearTimeout(this.frameInterval);
          this.frameInterval = null;
          return;
        }

        player.currentTime(nextFrameTime);
        this.frameInterval = setTimeout(playNextFrame, (1000 / this.fps) * 4);
      };

      playNextFrame();
    },
  },
};
</script>

<style>
.vjs-text-track-display {
  position: relative;
}
.custom-player-controls {
  font-size: medium;
  z-index: 100;
  position: absolute;
  top: 800px;
  color: black;
}
.video-js .vjs-tech {
  width: 1000px;
}
.video-js {
  width: 1000px;
}
</style>
