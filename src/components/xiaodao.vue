
<template>
  <div class="bodymain">
    <div id="threes" class="canves"></div>
  </div>
</template>
<script>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js'
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
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

var scene,camera,geometry,material,sky,orbitControls,renderer,watergeo,water,gltfloader;
export default {
  data() {
    return {
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      scene = new THREE.Scene()

      camera = new THREE.PerspectiveCamera(45, window.innerWidth/window.innerHeight,10, 1000)
      camera.position.set(10,10,130)
      camera.aspect = window.innerWidth/window.innerHeight
      camera.updateProjectionMatrix()
      scene.add(camera)

      renderer = new THREE.WebGLRenderer({
        // antialias: true
      })
      // renderer.outputEncoding = THREE.sRGBEncoding;
      renderer.setSize(window.innerWidth,window.innerHeight)
      document.getElementById('threes').appendChild(renderer.domElement)
      orbitControls = new OrbitControls(camera, renderer.domElement)
      this.renders()

      geometry = new THREE.SphereGeometry(550, 50, 50)
      let skyvideo = document.createElement('video')
      skyvideo.src = '/sky.mp4'
      skyvideo.loop = true;
      window.addEventListener('click', e => {
        if (skyvideo.pause) {
          skyvideo.play()
        }
      })
      material = new THREE.MeshBasicMaterial({
        map: new THREE.VideoTexture(skyvideo)
      })
      // material.map.needsUpdate = true
      geometry.scale(1,1,-1)
      sky = new THREE.Mesh(geometry, material)
      scene.add(sky)

      watergeo = new THREE.CircleGeometry(300, 10)
      water = new Water(watergeo, {
        textureWidth: 1024,
        textureHeight: 1024,
        color: 0xeeeeff,
        flowDirection: new THREE.Vector2(-1, 1),
        scale: 2,
        // normals: new THREE.TextureLoader().load('/waternormals.jpg', tex => {
        //   tex.wrapS = tex.wrapT = THREE.RepeatWrapping
        // }),
        sunDirection: new THREE.Vector3(),
        sunColor: 0xffffff,
      })
      water.rotation.x = -Math.PI / 2
      scene.add(water)

      // 添加环境光-没有方向
      const light = new THREE.DirectionalLight('#eeeeee', 1.0)
      gltfloader = new GLTFLoader()
      let draco = new DRACOLoader()
      gltfloader.setDRACOLoader(draco)
      gltfloader.load('/xd.glb', gltf => {
        var model = gltf.scene
        light.target = model
        scene.add(model)
      })
      scene.add(light)

    },
    renders() {
      orbitControls.update()
      renderer.render(scene, camera)
      requestAnimationFrame(this.renders)
      // this.water.material.uniforms['time'].value += 1.0/60.0
    }
  }
}
</script>

<style lang="scss">
.bodymain {
  width:100%;
  height: 100%;
  .main {
    width: 100%;
    height: 100%;
    background-color: aqua;
  }
  .canves{
    width: 200px;
    height: 200px;
    /* position: absolute; */
  }
}
</style>
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        