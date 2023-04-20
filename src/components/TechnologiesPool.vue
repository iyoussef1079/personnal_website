<template>
  <div id="technologies" ref="technologies">
    <div
      v-for="(technology, index) in technologies"
      :key="index"
      :style="{
        left: technology.x + 'px',
        top: technology.y + 'px',
        backgroundColor: technology.color,
      }"
      class="technology-ball"
    >
      {{ technology.name }}
    </div>
  </div>
</template>

<script>
import { Engine, World, Bodies, Body, Events } from "matter-js";

export default {
  name: "TechnologiesPool",
  components: {},
  props: {
    technologies: {
      type: Array,
      required: true,
    },
    ballSize: {
      type: Number,
      required: true,
      default: 100,
    },
  },
  data: function () {
    return {
      engine: null,
    };
  },
  methods: {
    initializeBalls() {
      const technologiesDiv = this.$refs.technologies;
      const rect = technologiesDiv.getBoundingClientRect();

      this.technologies.forEach((technology) => {
        const x = Math.random() * (rect.width - this.ballSize);
        const y = Math.random() * (rect.height - this.ballSize);
        technology.x = x;
        technology.y = y;

        const vx = Math.random() * 4 - 1;
        const vy = Math.random() * 4 - 1;

        // Create Matter.js circular body for the ball
        technology.body = Bodies.circle(
          x + this.ballSize / 2,
          y + this.ballSize / 2,
          this.ballSize / 2,
          {
            restitution: 0.8,
            friction: 0, // Set friction to 0
            frictionAir: 0,
          }
        );

        Body.setVelocity(technology.body, { x: vx, y: vy });
      });
    },
    setupMatter() {
      this.engine = Engine.create();
      this.engine.world.gravity.y = 0;

      // Get the dimensions of the #technologies div
      const technologiesDiv = this.$refs.technologies;
      const rect = technologiesDiv.getBoundingClientRect();

      // Create the walls (boundaries)
      const wallOptions = {
        isStatic: true,
      };
      const leftWall = Bodies.rectangle(
        -this.ballSize / 2,
        rect.height / 2,
        this.ballSize,
        rect.height,
        wallOptions
      );
      const rightWall = Bodies.rectangle(
        rect.width + this.ballSize / 2,
        rect.height / 2,
        this.ballSize,
        rect.height,
        wallOptions
      );
      const topWall = Bodies.rectangle(
        rect.width / 2,
        -this.ballSize / 2,
        rect.width,
        this.ballSize,
        wallOptions
      );
      const bottomWall = Bodies.rectangle(
        rect.width / 2,
        rect.height + this.ballSize / 2,
        rect.width,
        this.ballSize,
        wallOptions
      );

      // Add the balls to the Matter.js world
      const bodies = this.technologies.map((technology) => technology.body);
      World.add(this.engine.world, [
        ...bodies,
        leftWall,
        rightWall,
        topWall,
        bottomWall,
      ]);

      // Add a collision event listener
      Events.on(this.engine, "collisionStart", (event) => {
        event.pairs.forEach((pair) => {
          const { bodyA, bodyB } = pair;

          this.technologies.forEach((technology) => {
            if (technology.body === bodyA || technology.body === bodyB) {
              technology.colliding = true;
            }
          });
        });
      });

      // Run the Matter.js engine
      Engine.run(this.engine);

      // Update the positions of the balls in the Vue.js data
      this.updateBalls();
    },
    updateBalls() {
      this.technologies.forEach((technology) => {
        technology.x = technology.body.position.x - this.ballSize / 2;
        technology.y = technology.body.position.y - this.ballSize / 2;

        // Change the color of the balls based on the colliding property
        technology.color = technology.colliding ? "blue" : "red";
        technology.colliding = false;

        // Get the current velocity of the ball
        const velocity = technology.body.velocity;

        // Check if the velocity is below the threshold
        const threshold = 2;
        if (
          Math.abs(velocity.x) < threshold ||
          Math.abs(velocity.y) < threshold
        ) {
          // Apply a force to the ball to keep it moving
          const forceMagnitude = 0.005;
          Body.applyForce(technology.body, technology.body.position, {
            x: forceMagnitude * (Math.random() * 6 - 1),
            y: forceMagnitude * (Math.random() * 6 - 1),
          });
        }
      });

      this.$nextTick(() => {
        requestAnimationFrame(this.updateBalls);
      });
    },
  },
  mounted() {
    this.initializeBalls();
    this.setupMatter();
  },
};
</script>

<style>
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
  color: white;
  font-size: 14px;
  width: 100px;
  height: 100px;
}
</style>
