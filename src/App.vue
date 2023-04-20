<template>
  <body @mousemove="handleMouseMoove">
    <div id="blob" ref="blob"></div>
    <div id="blur" ref="blur"></div>
    <div id="content" ref="content">
      <div id="infoCard" ref="infoCard"></div>
      <technologies-pool
        :technologies="technologies"
        :ballSize="ballSize"
      ></technologies-pool>
    </div>
  </body>
</template>

<script>
import TechnologiesPool from "@/components/TechnologiesPool";

export default {
  name: "App",
  components: { TechnologiesPool },
  data: function () {
    return {
      ballSize: 100,
      technologies: [
        { name: "HTML", x: 0, y: 0, vx: 0, vy: 0 },
        { name: "CSS", x: 0, y: 0, vx: 0, vy: 0 },
        // Add more technologies here
      ],
    };
  },
  methods: {
    handleMouseMoove(event) {
      this.moveBlob(event);
      this.move3dCard(event);
    },
    moveBlob(event) {
      const blobElement = this.$refs.blob;

      // Update the position of the blob div based on the mouse coordinates
      blobElement.style.left =
        event.clientX - blobElement.clientWidth / 2 + "px";
      blobElement.style.top =
        event.clientY - blobElement.clientHeight / 2 + "px";

      blobElement.animate(
        {
          left: `${event.clientX - blobElement.clientWidth / 2}px`,
          top: `${event.clientY - blobElement.clientHeight / 2}px`,
        },
        {
          duration: 3000,
          fill: "forwards",
        }
      );
    },
    move3dCard(event) {
      const infoCardElement = this.$refs.infoCard;
      const xAxisRotation = (event.clientY / window.innerHeight - 0.5) * -30;
      const yAxisRotation = (event.clientX / window.innerWidth - 0.5) * 30;

      infoCardElement.style.transform = `perspective(1000px) rotateX(${xAxisRotation}deg) rotateY(${yAxisRotation}deg)`;
    },
  },
  async mounted() {
    this.initializeBalls();
    this.moveBalls();
  },
};
</script>

<style>
body {
  background-color: rgba(28, 28, 30, 255);
  min-height: 100vh;
  margin: 0;
  overflow: auto;
}

@keyframes rotate {
  from {
    rotate: 0;
  }

  50% {
    scale: 1 1.5;
  }

  to {
    rotate: 360deg;
  }
}

#blob {
  background: linear-gradient(to right, aquamarine, mediumpurple);
  height: 500px;
  aspect-ratio: 1;
  position: fixed;
  left: 50%;
  top: 50%;
  border-radius: 50%;
  animation: rotate 20s infinite;
}

#blur {
  height: 100%;
  width: 100%;
  position: fixed;
  z-index: 2;
  backdrop-filter: blur(200px);
}

#content {
  height: 100%;
  width: 100%;
  position: relative;
  z-index: 3;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-around;
}

#infoCard {
  position: relative;
  background-color: aliceblue;
  height: 600px;
  aspect-ratio: 1;
  transition: transform 0.1s;
  border-radius: 10%;
  background: url("./assets/photo_youssef.png");
  background-size: cover;
  margin-top: 10vh;
}
</style>
