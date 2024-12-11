<template>
  <div class="container">
    <h3>canvas</h3>

    <!-- 비디오 재생 -->
    <video ref="video" controls>
      <source src="/video/tiny.mp4" type="video/mp4" />
      이 브라우저는 비디오 태그를 지원하지 않습니다.
    </video>

    <canvas
      ref="canvas"
      @mousedown="startBoxing"
      @mouseup="endBoxing"
      @mousemove="drawBox"
    ></canvas>

    <div class="button-group">
      <button @click="startPlayback">일반 재생</button>
      <button @click="startFrameByFramePlayback">
        프레임 단위 재생(수정해야함)
      </button>
      <button @click="goToNextFrame">다음 프레임으로 이동</button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      frameRate: 30, // fps
      intervalId: null,
      isDrawing: false,
      startX: 0,
      startY: 0,
      endX: 0,
      endY: 0,
      data: [],
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
      console.log(videoElement.currentTime);

      if (videoElement.readyState >= 2) {
        const currentTime = videoElement.currentTime;
        const nextFrameTime = currentTime + 1 / this.frameRate;

        // 동영상 끝에 도달하지 않았으면 다음 프레임으로 이동
        if (nextFrameTime <= videoElement.duration) {
          videoElement.currentTime = nextFrameTime; // 다음 프레임 시간 설정
          this.captureFrame(videoElement); // 캔버스에 프레임 그리기
        } else {
          console.log("동영상 끝.");
        }
      } else {
        console.log("비디오 로드 중...");
      }
    },

    startBoxing(e) {
      this.isDrawing = true;
      const rect = this.$refs.canvas.getBoundingClientRect();
      const borderWidth = parseFloat(
        getComputedStyle(this.$refs.canvas).borderLeftWidth
      );
      const borderHeight = parseFloat(
        getComputedStyle(this.$refs.canvas).borderTopWidth
      );

      console.log("width: ", borderWidth, "height : ", borderHeight);
      this.startX = e.clientX - rect.left - borderWidth;
      this.startY = e.clientY - rect.top - borderHeight;

      console.log(this.startX);
    },

    endBoxing(e) {
      if (!this.isDrawing) return;

      const rect = this.$refs.canvas.getBoundingClientRect();
      const borderWidth = parseFloat(
        getComputedStyle(this.$refs.canvas).borderLeftWidth
      );
      const borderHeight = parseFloat(
        getComputedStyle(this.$refs.canvas).borderTopWidth
      );

      this.endX = e.clientX - rect.left - borderWidth;
      this.endY = e.clientY - rect.top - borderHeight;

      this.isDrawing = false;

      const videoElement = this.$refs.video;
      const currentTime = videoElement.currentTime;
      const frames = Math.round(currentTime * this.frameRate);

      const box = {
        startX: this.startX,
        startY: this.startY,
        endX: this.endX,
        endY: this.endY,
        frame: frames,
        time: currentTime,
      };
      // console.log("box info : ", box);
      this.data.push(box);
      // console.log("total data : ", this.data);

      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");

      ctx.strokeStyle = "red";
      ctx.lineWidth = 2;
      ctx.strokeRect(
        this.startX,
        this.startY,
        this.endX - this.startX,
        this.endY - this.startY
      );
      ctx.font = "16px Arial";
      ctx.fillStyle = "red";
      ctx.fillText("Label", this.startX + 5, this.startY - 5);
    },

    drawBox(e) {
      if (!this.isDrawing) {
        return;
      }

      const borderWidth = parseFloat(
        getComputedStyle(this.$refs.canvas).borderLeftWidth
      );
      const borderHeight = parseFloat(
        getComputedStyle(this.$refs.canvas).borderTopWidth
      );
      const rect = this.$refs.canvas.getBoundingClientRect();

      const currentX = e.clientX - rect.left - borderWidth;
      const currentY = e.clientY - rect.top - borderHeight;

      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");

      this.captureFrame(this.$refs.video);

      ctx.strokeStyle = "blue";
      ctx.lineWidth = 1;
      ctx.strokeRect(
        this.startX,
        this.startY,
        currentX - this.startX,
        currentY - this.startY
      );
    },
  },
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  max-width: 800px;
  margin: 0 auto;
}

h3 {
  font-size: 1.5rem;
  color: #333;
}

video {
  width: 100%;
  max-width: 800px;
  border-radius: 8px;
  border: 2px solid #ccc;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

canvas {
  border: 2px solid #ccc;
  border-radius: 8px;
  width: 100%;
  max-width: 800px;
  height: auto;
  background-color: #fff;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.button-group {
  display: flex;
  gap: 10px;
  justify-content: center;
}

button {
  padding: 10px 20px;
  font-size: 1rem;
  color: #fff;
  background-color: #007bff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #0056b3;
}

button:active {
  background-color: #003f7f;
}
</style>
