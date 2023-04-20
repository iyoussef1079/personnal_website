<template>
  <body @mousemove="handleMouseMoove">
    <div id="blob" ref="blob"></div>
    <div id="blur" ref="blur"></div>
    <div id="content" ref="content">
      <div id="infoCard" ref="infoCard"></div>
      <div id="technologies" ref="technologies">
        <div
          v-for="(technology, index) in technologies"
          :key="index"
          :style="{
            left: technology.x + 'px',
            top: technology.y + 'px',
          }"
          class="technology-ball"
        >
          {{ technology.name }}
        </div>
      </div>
    </div>
  </body>
</template>

<script>
export default {
  name: "App",
  components: {},
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
    checkCollision(ball1, ball2) {
      const dx = ball1.x - ball2.x;
      const dy = ball1.y - ball2.y;
      const distance = Math.sqrt(dx * dx + dy * dy);

      return distance < this.ballSize;
    },
    initializeBalls() {
      const technologiesDiv = this.$refs.technologies;
      const rect = technologiesDiv.getBoundingClientRect();

      this.technologies.forEach((technology, index) => {
        let positionValid;

        do {
          positionValid = true;
          technology.x = Math.random() * (rect.width - this.ballSize);
          technology.y = Math.random() * (rect.height - this.ballSize);

          // Check for collisions with other balls
          for (let i = 0; i < index; i++) {
            if (this.checkCollision(technology, this.technologies[i])) {
              positionValid = false;
              break;
            }
          }
        } while (!positionValid);

        technology.vx = Math.random() * 4 - 2; // Random velocity between -2 and 2
        technology.vy = Math.random() * 4 - 2; // Random velocity between -2 and 2
      });
    },
    moveBall() {
      const technologiesDiv = this.$refs.technologies;
      const rect = technologiesDiv.getBoundingClientRect();

      this.technologies.forEach((technology) => {
        technology.x += technology.vx;
        technology.y += technology.vy;

        // Check for collisions with the walls
        if (technology.x < 0 || technology.x > rect.width - this.ballSize) {
          technology.vx = -technology.vx;
        }
        if (technology.y < 0 || technology.y > rect.height - this.ballSize) {
          technology.vy = -technology.vy;
        }

        // Check for collisions with other balls
        this.technologies.forEach((otherTechnology) => {
          if (technology !== otherTechnology) {
            if (this.checkCollision(technology, otherTechnology)) {
              // Swap velocities
              const tempVx = technology.vx;
              const tempVy = technology.vy;
              technology.vx = otherTechnology.vx;
              technology.vy = otherTechnology.vy;
              otherTechnology.vx = tempVx;
              otherTechnology.vy = tempVy;
            }
          }
        });
      });
    },
    moveBalls() {
      this.technologies.forEach((technology) => {
        this.moveBall(technology);
      });
      this.$nextTick(() => {
        requestAnimationFrame(this.moveBalls);
      });
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

#technologies {
  position: relative;
  width: 100%;
  height: 60vh;
  background-color: white;
  margin-top: 70vh;
}

.technology-ball {
  position: absolute;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  background-color: #f1c40f;
  color: white;
  font-size: 14px;
  width: 100px;
  height: 100px;
}
</style>
