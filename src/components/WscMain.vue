
<template>
  <div>
    <div class="login">
      <div id="card" class="card" :style="{backgroundImage: `url(${imgUrl})`,backgroundSize: '100%',backgroundRepeat: 'space'}">
      </div>
      <!-- <LoginPage id="card" class="card" :cardImg="cardImg"></LoginPage> -->
    </div>
    <div class="bodymain" ref="bodyMain">
      <div class="logo-top"><img src="../assets/vue.svg"><h2>葳蕤生香</h2></div>
      <div class="main-text" ref="pages">
        <div class="text-page">
          <h1>为天地立心，为生民立命，<br>
            为往圣继绝学，为万世开太平。</h1>
        </div>
        <div class="text-page">
          <h1>落霞与孤鹜齐飞</h1>
        </div>
        <div class="text-page">
          <h1>秋水共长天一色</h1>
        </div>
      </div>
      <!-- <div id="threes" class="canves"></div> -->
    </div>
  </div>
</template>
<script>
import LoginPage from './LoginPage.vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import Gsap from 'gsap';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer';
// 后处理渲染器
import { OutlinePass } from 'three/examples/jsm/postprocessing/OutlinePass';
// 后处理渲染通道
import {RenderPass} from 'three/examples/jsm/postprocessing/RenderPass'
// hrd加载器
import {RGBELoader} from 'three/examples/jsm/loaders/RGBELoader'
// draco解码器
import {DRACOLoader} from 'three/examples/jsm/loaders/DRACOLoader'
// 导入tween-补间动画
import * as TWEEN from 'three/examples/jsm/libs/tween.module'
import { Water } from 'three/examples/jsm/objects/Water2.js'

var scene,camera,geometry,material,sky,orbitControls,renderer,watergeo,water,gltfloader,channel,cardpage;
export default {
  data() {
    return {
      imgUrl: ''
    }
  },
  components: {LoginPage},
  created() {
    this.initCard()
  },
  mounted() {
    this.init()
    channel = new BroadcastChannel('card')
    channel.onmessage = (e) => {
      const [x, y] = this.screenToClient(...e.data)
      cardpage.style.left = x + 'px';
      cardpage.style.top = y + 'px';
    }
    cardpage = document.querySelector('#card')
    cardpage.onmousedown = (e) => {
      let x = e.pageX - cardpage.offsetLeft;
      let y = e.pageY - cardpage.offsetTop;
      window.onmousemove = (es) => {
        const cx = es.pageX - x;
        const cy = es.pageY - y;
        cardpage.style.left = cx + 'px';
        cardpage.style.top = cy + 'px';
        const point = this.clientToScreen(cx, cy) // 视口转屏幕
        channel.postMessage(point)
      };
      window.onmouseup = () => {
        window.onmousemove = null;
        window.onmouseup = null;
      }
    }
  },
  methods: {
    // 浏览器导航条高度
    barHeight() {
      return window.outerHeight - window.innerHeight
    },
    // 视口坐标转化浏览器坐标
    clientToScreen(clientX, clientY) {
      const screenX = clientX + window.screenX
      const screenY = clientY + window.screenY + this.barHeight()
      return [screenX, screenY]
    },
    screenToClient(screenX, screenY) {
      const clientX = screenX - window.screenX 
      const clientY = screenY - window.screenY - this.barHeight()
      return [clientX, clientY]
      
    },
    // form标签可拖动
    initCard() {
      const url = new URL(window.location.href);
      const type = url.searchParams.get("type");
      this.imgUrl = new URL(`../assets/${type}.jpg`, import.meta.url).href
    },
    init() {
      scene = new THREE.Scene();
      camera = new THREE.PerspectiveCamera(45,window.innerWidth/window.innerHeight,0.1,1000);
      camera.position.set(0,0,10);
      renderer = new THREE.WebGLRenderer({antialias: true});
      renderer.setSize(window.innerWidth,window.innerHeight);
      renderer.setClearColor(0x000000);

      this.$refs['bodyMain'].appendChild(renderer.domElement);
      
      let url = '/25s.jpg';
      let envTexture = new THREE.TextureLoader().load(url);
      envTexture.mapping = THREE.EquirectangularReflectionMapping;
      scene.background = envTexture;
      scene.environment = envTexture
      

      // const rgbeload = new RGBELoader()
      // rgbeload.load('/HDRMAP.hdr', hdr => {
      //   hdr.mapping = THREE.EquirectangularReflectionMapping;
      //   scene.background = hdr;
      //   scene.environment = hdr
      // })
      this.renderFnc()
      // 添加环境光-没有方向
      const light1 = new THREE.DirectionalLight('#fff', 1.0)
      light1.position.set(1,1,1)
      scene.add(light1)
      const light2 = new THREE.DirectionalLight('#fff', 1.0)
      light2.position.set(-1,-1,-1)
      scene.add(light2)
      const light3 = new THREE.DirectionalLight('#fff', 1.0)
      light3.position.set(-1,1,1)
      scene.add(light3)
      const light4 = new THREE.DirectionalLight('#fff', 1.0)
      light4.position.set(1,1,-1)
      scene.add(light4)
      gltfloader = new GLTFLoader()
      gltfloader.load('/fj.gltf', e => {
        let model = e.scene
        model.scale.set(0.2,0.2,0.2)
        model.rotateY(0.2)
        model.rotateX(-0.2)
        model.position.x = 5
        // light1.target = model
        // light2.target = model
        // light3.target = model
        // light4.target = model

        window.addEventListener('mousemove', e => {
          const x = (e.clientX/window.innerWidth) * 2 - 1
          const y = (e.clientY/window.innerHeight) * 2 - 1
          let gsap = Gsap.timeline()
          gsap.to(model.rotation, {
            duration: 1,
            x: y,
            y: x,
          })
        })
        scene.add(model)
      })
      gltfloader.load('/as/carred1.gltf', e => {
        let model = e.scene
        model.rotateY(0.2)
        model.rotateX(-0.2)
        model.position.x = 5
        model.position.y = -8
        light1.target = model
        light2.target = model
        light3.target = model
        light4.target = model

        window.addEventListener('mousemove', e => {
          const x = (e.clientX/window.innerWidth) * 2 - 1
          const y = (e.clientY/window.innerHeight) * 2 - 1
          let gsap = Gsap.timeline()
          gsap.to(model.rotation, {
            duration: 1,
            x: y,
            y: x,
          })
        })
        scene.add(model)
      })
      gltfloader.load('/lc.gltf', e => {
        let model = e.scene
        model.scale.set(0.1,0.1,0.1)
        model.rotateY(0.2)
        model.rotateX(-0.2)
        model.position.x = 5
        model.position.y = -16
        light1.target = model
        light2.target = model
        light3.target = model
        light4.target = model

        window.addEventListener('mousemove', e => {
          const x = (e.clientX/window.innerWidth) * 2 - 1
          const y = (e.clientY/window.innerHeight) * 2 - 1
          let gsap = Gsap.timeline()
          gsap.to(model.rotation, {
            duration: 1,
            x: y,
            y: x,
          })
        })
        scene.add(model)
      })
      
      let page = 0
      let timeline2 = Gsap.timeline()
      window.addEventListener('mousewheel', e => {
        if (e.wheelDelta < 0) {
          page++;
          if (page > 2) {
            page = 2
          }
        }
        if (e.wheelDelta > 0) {
          page--;
          if (page < 0) {
            page = 0
          }
        }
        if (!timeline2.isActive()) {
          timeline2.to(camera.position, {
            duration: 1,
            y: page * -8,
          })
          Gsap.to(this.$refs['pages'],{
            duration: 1,
            y: -page * window.innerHeight
          })
        }
      })
      let dracoLoader = new DRACOLoader();
      dracoLoader.setDecoderPath("/draco/gltf/");
      dracoLoader.setDecoderConfig({ type: "js" });
      gltfloader.setDRACOLoader(dracoLoader);
      gltfloader.load("/moon.glb", gltf => {
        let moon = gltf.scene.children[0]
        for (let j = 0; j < 10; j++) {
          let moonInstance = new THREE.InstancedMesh( // 实例化mesh
            moon.geometry,
            moon.material,
            100
          )
          for (let i = 0; i < 100; i++) {
            let x = Math.random() * 1000 - 500
            let y = Math.random() * 1000 - 500
            let z = Math.random() * 1000 - 500

            let matrix = new THREE.Matrix4()
            let size = Math.random() * 20 - 8;
            matrix.makeScale(size, size, size);
            matrix.makeTranslation(x,y,z)
            moonInstance.setMatrixAt(i, matrix)
          }
          Gsap.to(moonInstance.position, {
            duration: Math.random() * 10 + 2,
            z: -1000,
            ease: "linear",
            repeat: -1,
          })
          scene.add(moonInstance)
        }
      })
    },
    renderFnc() {
      requestAnimationFrame(this.renderFnc)
      renderer.render(scene,camera)
    }
  }
}
</script>

<style lang="scss">
  .login {
    position: relative;
    left: 35%;
    z-index: 2;
    .card {
      width: 200px;
      height: 400px;
      position: absolute;
      cursor: pointer;
    }
  }
.bodymain {
  width:100%;
  height: 100%;
  transform-origin: 0 0;
  .main {
    width: 100%;
    height: 100%;
    background-color: aqua;
  }
  .logo-top{
    position: absolute;
    display: flex;
    color: rgb(187, 240, 253);
    top: 5vh;
    left: 5vw
  }
  .main-text{
    display: flex;
    flex-direction: column;
    position: fixed;
    top: 0;
    left: 0;
    .text-page {
      width: 100vw;
      height: 100vh;
      justify-content: center;
      align-items: flex-start;
      color: #fff;
      position: relative;
      left: 10vw;
      top: 40vh;
      box-sizing: border-box;
    }
  }
}
</style>