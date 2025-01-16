<template>
  <div
    style="
      width: 100vw;
      height: 100vh;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
    "
  >
    <div ref="threeContainer" style="width: 100%; height: 90%"></div>
    <button
      :disabled="isRolling"
      @click="rollDice"
      style="
        margin-top: 20px;
        padding: 10px 20px;
        font-size: 16px;
        background-color: rgba(255, 255, 255, 0.5);
        border: none;
        border-radius: 5px;
        color: #000;
        cursor: pointer;
        opacity: 0.8;
      "
      :style="isRolling ? 'cursor: not-allowed; opacity: 0.5;' : ''"
    >
      Вращать кубики
    </button>
  </div>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

export default {
  name: "TwoDice",
  data() {
    return {
      isRolling: false,
      initialRotationSpeed: { x: 0.08, y: 0.08, z: 0.05 },
      diceRotationSpeed: { x: 0.08, y: 0.08, z: 0.05 },
    };
  },
  mounted() {
    this.initScene();
  },
  methods: {
    scene: null,
    camera: null,
    renderer: null,
    dice1: null,
    dice2: null,

    initScene() {
      const container = this.$refs.threeContainer;
      this.scene = new THREE.Scene();

      this.camera = new THREE.PerspectiveCamera(
        50,
        container.clientWidth / container.clientHeight,
        0.1,
        1000
      );
      this.camera.position.z = 6;

      this.renderer = new THREE.WebGLRenderer({ antialias: true });
      this.renderer.setSize(container.clientWidth, container.clientHeight);
      this.renderer.setPixelRatio(window.devicePixelRatio);
      container.appendChild(this.renderer.domElement);

      this.createGradientBackground();

      const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
      this.scene.add(ambientLight);

      const directionalLight1 = new THREE.DirectionalLight(0xffffff, 0.6);
      directionalLight1.position.set(-5, 5, 5);
      this.scene.add(directionalLight1);

      const directionalLight2 = new THREE.DirectionalLight(0xffffff, 0.6);
      directionalLight2.position.set(5, 5, 5);
      this.scene.add(directionalLight2);

      const loader = new GLTFLoader();
      loader.load(
        "/models/Dice_03.glb",
        (gltf) => {
          this.dice1 = gltf.scene.clone(true);
          this.dice2 = gltf.scene.clone(true);

          this.positionDice(this.dice1, -1.5);
          this.positionDice(this.dice2, 1.5);

          this.scene.add(this.dice1);
          this.scene.add(this.dice2);

          this.animate();
        },
        undefined,
        (error) => {
          console.error("Ошибка загрузки модели:", error);
        }
      );

      window.addEventListener("resize", this.onWindowResize);
    },

    createGradientBackground() {
      const texture = new THREE.CanvasTexture(this.generateGradient());
      const geometry = new THREE.PlaneGeometry(20, 20);
      const material = new THREE.MeshBasicMaterial({ map: texture });
      const plane = new THREE.Mesh(geometry, material);
      plane.position.z = -10;
      this.scene.add(plane);
    },

    generateGradient() {
      const canvas = document.createElement("canvas");
      canvas.width = 512;
      canvas.height = 512;
      const ctx = canvas.getContext("2d");
      const gradient = ctx.createRadialGradient(
        canvas.width / 2,
        canvas.height / 2,
        50,
        canvas.width / 2,
        canvas.height / 2,
        canvas.width / 2
      );
      gradient.addColorStop(0, "#031706");
      gradient.addColorStop(1, "#000");
      ctx.fillStyle = gradient;
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      return canvas;
    },

    positionDice(dice, xPosition) {
      const box = new THREE.Box3().setFromObject(dice);
      const center = box.getCenter(new THREE.Vector3());
      const size = box.getSize(new THREE.Vector3()).length();
      dice.position.set(xPosition - center.x, -center.y, -center.z);
      const scale = 2 / size;
      dice.scale.set(scale, scale, scale);
      dice.rotation.set(0, 0, 0);
    },

    animate() {
      requestAnimationFrame(() => this.animate());
      this.renderer.render(this.scene, this.camera);
    },

    onWindowResize() {
      const container = this.$refs.threeContainer;
      this.camera.aspect = container.clientWidth / container.clientHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(container.clientWidth, container.clientHeight);
    },

    rollDice() {
      if (this.isRolling || !this.dice1 || !this.dice2) return;

      this.isRolling = true;

      // Сброс скорости вращения
      this.diceRotationSpeed = { ...this.initialRotationSpeed };

      const duration = 3000;
      const stopTime = 1000;
      const startTime = performance.now();

      const animateRotation = (time) => {
        const elapsed = time - startTime;

        if (elapsed < duration) {
          this.dice1.rotation.x += this.diceRotationSpeed.x;
          this.dice1.rotation.y += this.diceRotationSpeed.y;
          this.dice1.rotation.z += this.diceRotationSpeed.z;

          this.dice2.rotation.x += this.diceRotationSpeed.x;
          this.dice2.rotation.y += this.diceRotationSpeed.y * 1.2;
          this.dice2.rotation.z += this.diceRotationSpeed.z;

          if (elapsed > stopTime) {
            const factor = (duration - elapsed) / (duration - stopTime);
            this.diceRotationSpeed.x = this.initialRotationSpeed.x * factor;
            this.diceRotationSpeed.y = this.initialRotationSpeed.y * factor;
            this.diceRotationSpeed.z = this.initialRotationSpeed.z * factor;
          }

          requestAnimationFrame(animateRotation);
        } else {
          this.isRolling = false;
        }
      };

      requestAnimationFrame(animateRotation);
    },
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.onWindowResize);
  },
};
</script>

<style>
body {
  margin: 0;
  overflow: hidden;
}
button {
  cursor: pointer;
}
</style>
