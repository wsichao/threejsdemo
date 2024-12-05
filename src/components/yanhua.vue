<template>
  <canvas ref="canvas" class="fireworks-canvas"></canvas>
</template>

<script>
export default {
  name: "Fireworks",
  data() {
    return {
      canvas: null,
      ctx: null,
      fireworks: [],
      fireworkColors: ["#ff4500", "#ff6347", "#32cd32", "#1e90ff", "#8a2be2", "#ff1493"],
    };
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.ctx = this.canvas.getContext("2d");
    this.resizeCanvas();
    window.addEventListener("resize", this.resizeCanvas);
    this.startFireworks();
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.resizeCanvas);
  },
  methods: {
    resizeCanvas() {
      this.canvas.width = window.innerWidth;
      this.canvas.height = window.innerHeight;
    },
    startFireworks() {
      setInterval(this.createFirework, 1000); // 每秒钟创建一个新的烟花
      this.animate();
    },
    createFirework() {
      const x = Math.random() * this.canvas.width;
      const y = this.canvas.height; // 烟花从底部发射
      this.fireworks.push({
        x,
        y,
        particles: this.generateParticles(x, y),
      });
    },
    generateParticles(x, y) {
      const particles = [];
      const count = Math.floor(Math.random() * 50 + 100); // 随机粒子数量
      const startColor = this.fireworkColors[Math.floor(Math.random() * this.fireworkColors.length)];
      const endColor = this.fireworkColors[Math.floor(Math.random() * this.fireworkColors.length)];

      for (let i = 0; i < count; i++) {
        const angle = (Math.PI * 2 * i) / count;
        const speed = Math.random() * 2 + 1;

        // 创建发射粒子，粒子开始向上运动
        particles.push({
          x,
          y,
          vx: Math.cos(angle) * speed,
          vy: Math.sin(angle) * speed - 5,  // 通过减少y轴速度，让粒子先向上发射
          radius: Math.random() * 2 + 1,
          life: Math.random() * 60 + 30,  // 初始寿命
          opacity: 1,
          startColor, // 起始颜色
          endColor, // 结束颜色
          gradientProgress: 0, // 渐变进度
          isExploded: false, // 是否已经爆炸
        });
      }

      return particles;
    },
    drawFireworks() {
      this.ctx.fillStyle = "rgba(0, 0, 0, 0.1)";
      this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);

      this.fireworks.forEach((firework, fireworkIndex) => {
        firework.particles.forEach((particle, particleIndex) => {
          // 更新粒子位置
          if (!particle.isExploded) {
            // 发射阶段：粒子逐渐减速
            particle.vy += 0.05;  // 模拟重力效果
          } else {
            // 爆炸阶段：粒子在各个方向扩散
            particle.x += particle.vx;
            particle.y += particle.vy;
          }

          // 当粒子到达最高点，触发爆炸
          if (!particle.isExploded && particle.vy > 0) {
            particle.isExploded = true; // 开始爆炸
            this.explodeFirework(particle);  // 生成爆炸效果
          }

          // 渲染粒子
          this.renderParticle(particle);

          // 更新粒子寿命
          particle.opacity -= 0.01;
          particle.life--;

          // 移除粒子
          if (particle.opacity <= 0 || particle.life <= 0) {
            firework.particles.splice(particleIndex, 1);
          }
        });

        // 移除空的烟花
        if (firework.particles.length === 0) {
          this.fireworks.splice(fireworkIndex, 1);
        }
      });
    },
    explodeFirework(particle) {
      // 产生爆炸效果的粒子
      const explosionParticles = this.generateParticles(particle.x, particle.y);
      // 将爆炸粒子添加到烟花中
      this.fireworks.push({
        x: particle.x,
        y: particle.y,
        particles: explosionParticles,
      });
    },
    renderParticle(particle) {
      const currentColor = this.interpolateColor(
        particle.startColor,
        particle.endColor,
        particle.gradientProgress
      );

      this.ctx.beginPath();
      this.ctx.arc(particle.x, particle.y, particle.radius, 0, Math.PI * 2);
      this.ctx.fillStyle = `rgba(${currentColor.r}, ${currentColor.g}, ${currentColor.b}, ${particle.opacity})`;
      this.ctx.fill();
    },
    interpolateColor(startColor, endColor, progress) {
      const startRGB = this.hexToRgb(startColor);
      const endRGB = this.hexToRgb(endColor);
      return {
        r: Math.floor(startRGB.r + (endRGB.r - startRGB.r) * progress),
        g: Math.floor(startRGB.g + (endRGB.g - startRGB.g) * progress),
        b: Math.floor(startRGB.b + (endRGB.b - startRGB.b) * progress),
      };
    },
    hexToRgb(hex) {
      const bigint = parseInt(hex.slice(1), 16);
      return {
        r: (bigint >> 16) & 255,
        g: (bigint >> 8) & 255,
        b: bigint & 255,
      };
    },
    animate() {
      this.drawFireworks();
      requestAnimationFrame(this.animate);
    },
  },
};
</script>

<style scoped>
.fireworks-canvas {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 9999;
  background-color: black;
}
</style>
<!-- ，添加声音效果，添加鼠标交互，添加3D效果，添加烟雾效果，添加动态背景，添加多层次爆炸，添加涂鸦模式，添加色彩交替，添加爆炸后形成动态图案 -->