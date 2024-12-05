
<template>
  <div class="bodymain">
    <!-- <div class="main">
      <span @click="setColor">绿</span>
    </div> -->
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
// 导入tween
import * as TWEEN from 'three/examples/jsm/libs/tween.module'



export default {
  data() {
    return {

    }
  },
  mounted() {
    this.init()
    // this.setMoreGeo()
  },
  methods: {
    setRaycaster() {
      // 创建射线
      const raycaster = new THREE.Raycaster()
      // 创建鼠标向量
      const mouse = new THREE.Vector2()
      window.addEventListener('click', evn => {
        mouse.x = (evn.clientX / window.innerWidth) * 2 - 1
        mouse.y = -(evn.clientY / window.innerHeight) * 2 + 1
        // 通过摄像机和鼠标点击的位置获取射线
        raycaster.setFromCamera(mouse, this.camera)
        // 计算射线与物体的交点
        const intersects = raycaster.intersectObjects(this.scene.children)
        console.log(intersects);
        if (intersects.length > 0) {
          if (intersects[0].object._isSelect) {
            intersects[0].object.material.color.set(intersects[0].object._orginColor)
            intersects[0].object._isSelect = false
            return
          }
          intersects[0].object._isSelect = true
          intersects[0].object._orginColor = intersects[0].object.material.color.getHex()
          intersects[0].object.material.color.set(0xff0000)
        }
      })
    },
    initLoader() {
      let self = this
      const loader = new GLTFLoader();
      loader.load('/fj.gltf', function ( gltf ) {
        gltf.scene.rotateY(5)
        self.group.add(gltf.scene);
      }, function (xhr) {
        // 计算加载进度
        // const percent = xhr.loaded / xhr.total
        // console.log(percent);
      }, function ( error ) {
        console.error( error );
      } );

      const rgbeLoader = new RGBELoader()
      rgbeLoader.load('./public/10K.hdr', (envMap) => {
        // 设置球形贴图
        envMap.mapping = THREE.EquirectangularReflectionMapping
        // 环境贴图
        this.scene.background = envMap
        this.scene.environment = envMap
        self.group.traverse(function(obj){
          if(obj.isMesh) {
            obj.material.envMap = envMap
          }
        })
      })
      // 光线投影交互
      // this.setRaycaster()
      // 补间动画
      this.group.position.x = -4
      const tween = new TWEEN.Tween(this.group.position)
      tween.to({x: 54}, 15000)
      tween.repeat(Infinity)
      // tween.easing(TWEEN.Easing.Quadratic.InOut)
      tween.start()
    },
    creatMesh() {
      // 创建材质
      // let texturecube = new THREE.CubeTextureLoader().load(['../../public/no-message.png', '/public/no-message.png','/public/no-message.png', '/public/no-message.png','/public/no-message.png', '/public/no-message.png'], function ( gltf ) {
      //   console.log( gltf );
      // }, undefined, function ( error ) {
      //   console.error( error );
      // } )
      this.
      this.material = new THREE.MeshStandardMaterial({
        color: 'RED',
        envMap: texturecube,
        envMapIntensity: 1.0,
        transparent: true,
        metalness: 0.0,
        roughness: 0.1,
        transmission: 0.9,
        ior: 1.5
      })
    },
    initCamera() {
      this.camera = new THREE.PerspectiveCamera(60, this.width/this.height, 1, 4000)
      // 相机设置
      this.camera.position.set(15,20,25)
      this.camera.lookAt(0,200,200)
    },
    addFog() {
      const fogs = new THREE.FogExp2('#ccc', 0.1)
      // const fogs = new THREE.Fog('#000', 10, 50)
      this.scene.fog = fogs
      this.scene.background = new THREE.Color('#ccc')
    },
    init() {
      this.gui = new GUI()
      this.width = window.innerWidth
      this.height = window.innerHeight
      this.scene = new THREE.Scene();
      // 创建立方体
      // this.geometry = new THREE.BoxGeometry(150,150,150);
      this.group = new THREE.Group()
      // 初始化加载loader素材
      this.initLoader()

      this.scene.add(this.group)
      // 初始化相机
      this.initCamera()
      // 创建mesh
      // this.creatMesh()
      // 添加雾效果
      // this.addFog()
      this.renderer = new THREE.WebGLRenderer();
      // 设置背景透明度
      this.renderer.setClearAlpha(0.0);
      this.initThree()
      // this.renderer.render(this.scene, this.camera)
      // this.renderer.outputEncoding = THREE.sRGBEncoding
    },
    setColor() {
      this.group.traverse(item => {
        if (item.isMesh) {
          item.material.color.set('#444')
        }
      })
      this.renderer.render(this.scene, this.camera)
    },
    // 矩阵geo
    setMoreGeo() {
      // for (let x = 0; x < 10; x++) {
      //   for (let z = 0; z < 10; z++) {
      //     if (z >=5 && x>=5) {
      //       this.geometry = new THREE.SphereGeometry(50);
      //     }
      //     this.mesh = new THREE.Mesh(this.geometry, this.material)
      //     this.gui.add(this.mesh.position, 'x', 0, 1000)
      //     this.gui.add(this.mesh.position, 'y', 0, 1000)
      //     this.gui.add(this.mesh.position, 'z', 0, 1000)
      //     // 网格定位
      //     this.mesh.position.set(x * 200, 0, z * 200)
      //     this.scene.add(this.mesh)
      //     this.runDemo(this.mesh, this.scene, this.camera, this.renderer)
      //   }
      // }
      // this.mesh = new THREE.Mesh(this.geometry, this.material)
      // this.mesh.position.set(0, 0, 0)
      // this.scene.add(this.mesh)
    },
    initThree() {
      // 添加坐标轴
      const axesHelper = new THREE.AxesHelper(500)
      this.scene.add(axesHelper)

      // 添加点光源-四周发散
      // const pointLight = new THREE.PointLight("#fff", 1.0);
      // pointLight.decay = 0.0
      // pointLight.position.set(200, 200, 100)
      // 显示点光源辅助
      // const pointLightHelper = new THREE.PointLightHelper(pointLight, 20)
      // this.scene.add(pointLight)
      // this.gui.add(pointLight, 'intensity', 0, 1)
      // 添加环境光-没有方向
      // const ambient = new THREE.AmbientLight('#fff', 1.0)
      // this.scene.add(ambient)
      // this.gui.add(ambient, 'intensity', 0, 2)
      // 添加平行光
      // const directionalLight = new THREE.DirectionalLight('#fff', 1.0)
      // directionalLight.position.set(200,200,100)
      // directionalLight.target= this.group
      // this.gui.add(directionalLight, 'intensity', 0, 3)
      // 添加平行光辅助
      // const directionalLightHelper = new THREE.DirectionalLightHelper(directionalLight, 10)
      // this.scene.add(directionalLight)

      // 渲染设置
      document.getElementById('threes').appendChild(this.renderer.domElement)
      this.renderer.setSize(this.width, this.height)
      this.camera.updateProjectionMatrix()
      this.renderer.render(this.scene, this.camera)

      // const effectComposer = new EffectComposer(this.renderer)
      // const renderpass = new RenderPass(this.scene, this.camera)
      // effectComposer.addPass(renderpass)
      // const v2 = new THREE.Vector2(this.width, this.height)
      // const outlinePass = new OutlinePass(v2, this.scene, this.camera)
      // outlinePass.selectedObjects = [this.group]
      // effectComposer.addPass(outlinePass)
      // effectComposer.render()

      // 添加旋转缩放
      const controls = new OrbitControls(this.camera, this.renderer.domElement)
      controls.target.set(0,0,0)
      let self = this
      controls.addEventListener('change', function() {
        self.renderer.render(self.scene, self.camera)
      })
      controls.update()
      this.runDemo(controls)
      this.renderer.render(this.scene, this.camera)
      // this.runDemo(this.group, this.scene, this.camera, this.renderer)
    },
    // animate(controls) {
    //   controls.update()
    //   requestAnimationFrame(this.animate(controls))
    //   this.renderer.render(this.scene, this.camera)
    // },
    // 添加旋转动画
    runDemo(controls) {
      if (controls.update) {
        controls.update()
      }
      // this.group.rotateY(0.1)
      requestAnimationFrame(this.runDemo)
      this.renderer.render(this.scene, this.camera)
      TWEEN.update()
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
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        