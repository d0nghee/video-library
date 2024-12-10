<template>
  <div>
    <h3>canvas</h3>

    <!-- 비디오 재생 -->
    <video ref="video" controls>
      <source src="/video/tiny.mp4" type="video/mp4" />
      이 브라우저는 비디오 태그를 지원하지 않습니다.
    </video>

    <canvas ref="canvas" style="border: 1px solid black"></canvas>

    <button @click="startPlayback">일반 재생</button>

    <button @click="startFrameByFramePlayback">프레임 단위로 재생</button>

    <button @click="goToNextFrame">다음 프레임으로 이동</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      frameRate: 30, // fps
      intervalId: null,
    };
  },
  methods: {
    // 비디오의 현재 프레임을 캡처하고 캔버스에 그리기
    captureFrame(videoElement) {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");

      // 캔버스 크기 설정
      canvas.width = videoElement.videoWidth;
      canvas.height = videoElement.videoHeight;

      // 비디오 프레임을 캔버스에 그리기
      ctx.drawImage(videoElement, 0, 0, canvas.width, canvas.height);

      // 현재 프레임 정보 콘솔에 출력
      console.log(`현재 프레임: ${videoElement.currentTime.toFixed(2)}초`);
    },

    // 일반 비디오 재생 시작
    startPlayback() {
      const videoElement = this.$refs.video;
      videoElement.playbackRate = 1; // 기본 재생 속도
      if (videoElement.paused) {
        videoElement.play(); // 비디오 재생
      }
      // 프레임 단위 재생을 위한 setInterval을 정리
      if (this.intervalId) {
        clearInterval(this.intervalId);
        this.intervalId = null;
      }
    },

    // 프레임 단위로 비디오 재생 시작
    startFrameByFramePlayback() {
      const videoElement = this.$refs.video;
      videoElement.playbackRate = 0; // 재생 속도를 0으로 설정하여 수동 재생
      console.log(videoElement.playbackRate);
      // 비디오가 멈춰있으면 재생 시작
      if (videoElement.paused) {
        videoElement.play();
      }

      // 프레임 단위로 이동하는 setInterval 설정
      if (this.intervalId) {
        clearInterval(this.intervalId); // 기존 인터벌이 있다면 먼저 정리
      }

      this.intervalId = setInterval(() => {
        const currentTime = videoElement.currentTime;
        const nextFrameTime = currentTime + 1 / this.frameRate;

        // 비디오가 끝날 때까지 프레임 단위로 이동
        if (nextFrameTime <= videoElement.duration) {
          videoElement.currentTime = nextFrameTime;
          this.captureFrame(videoElement);
        } else {
          clearInterval(this.intervalId); // 동영상 끝나면 자동 종료
          console.log("동영상 끝.");
        }
      }, 1000 / this.frameRate); // 프레임당 시간 간격 (초단위)
    },

    // 버튼 클릭 시 다음 프레임으로 이동
    goToNextFrame() {
      const videoElement = this.$refs.video;
      console.log();

      // 현재 비디오가 로드되고 재생 중인지 확인
      if (!videoElement.paused) {
        const currentTime = videoElement.currentTime;
        const nextFrameTime = currentTime + 1 / this.frameRate;

        if (nextFrameTime <= videoElement.duration) {
          videoElement.currentTime = nextFrameTime;
          this.captureFrame(videoElement);
        } else {
          console.log("동영상 끝.");
        }
      }
    },
  },
};
</script>

<style scoped>
video {
  width: 100%;
  height: auto;
}
</style>
