<template>

</template>

<script setup>
import * as THREE from 'three'
import gsap from 'gsap'; // 补间动画
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader"
import { Water } from "three/examples/jsm/objects/Water2"
// 初始化场景 
const scene = new THREE.Scene();
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75, 
  window.innerWidth / window.innerHeight, 
  0.1,
  1000);
camera.position.set(-3.23,2.98,4.06)  
camera.updateProjectionMatrix() // 投影矩阵
  // 初始化渲染器
const renderer = new THREE.WebGLRenderer({
  antialias: true, // 抗锯齿
})
renderer.setSize(window.innerWidth, window.innerHeight)
document.body.appendChild(renderer.domElement)
// 设置色调映射
renderer.outputEncoding = THREE.sRGBEncoding
renderer.toneMapping = THREE.ACESFilmicToneMapping
renderer.toneMappingExposure = 1 // 默认就是1 
renderer.shadowMap.enabled = true // 允许阴影

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement)
controls.enableDamping = true // 阻尼效果

//解码和导入模型
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath('./draco/')
const gltfLoader = new GLTFLoader()
gltfLoader.setDRACOLoader(dracoLoader)

// 加载环境纹理
// hdr  先导包  RGBE
const rgbeLoader = new RGBELoader()
rgbeLoader.load("./textures/sky.hdr",texture => {
  texture.mapping = THREE.EquirectangularReflectionMapping // 全景图 环境映射
  scene.background = texture
  scene.environment = texture
})

// 加载模型
gltfLoader.load('./model/scene.glb',  (gltf) =>{
  const model = gltf.scene
  // 隐藏自带的水纹
  model.traverse((child) => {
    if(child.name === 'Plane') {
      child.visible = false
    }
    if(child.isMesh) {
      child.castShadow = true
      child.receiveShadow = true
    }
    })
  scene.add(model)
})

// 设置水面效果 
const waterGeometry = new THREE.CircleGeometry(300,32)
const water = new Water(waterGeometry, {
  textureWidth : 1024 ,
  textureHeight: 1024,
  color:0xeeeeff,
  flowDirection:new THREE.Vector2(1,1),
  scale:100,
  // alpha: 1.0,
  // sunDirection: new THREE.Vector3(0, -1, 0),
  // sunColor: 0xffffff,
  // waterColor: 0x001e0f,
})
scene.add(water)
water.rotation.x = -Math.PI / 2
water.position.y = -0.4

// 添加平行光
const light = new THREE.DirectionalLight(0xffffff, 0.5)
light.position.set(0, 50, 0)
scene.add(light)

// 添加点光源 (房子里面)
const planeLight = new THREE.PointLight(0xffffff,50)
planeLight.position.set(0.1, 2.4, 0)
planeLight.castShadow = true // 阴影
scene.add(planeLight)

//----------------------------------创建点光源组----------------------------------------
const pointLightGroup = new THREE.Group()
pointLightGroup.position.set(-8,2.5,-1.5)
const pointLightArr =  []
let radius;
for(let i = 0; i< 3; i++) {
  const sphereGeometry = new THREE.SphereGeometry(0.2,32,32)
  const sphereMaterial = new THREE.MeshBasicMaterial({ 
    color:0xffffff,
    emissive:0xffffff, // 亮度
    emissiveIntensity:10 // 强度
   })
  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial)
  pointLightArr.push(sphere)
  sphere.position.set(
    radius * Math.cos((i * 2 * Math.PI) / 3),
    Math.cos((i * 2 * Math.PI) / 3),
    radius * Math.sin((i * 2 * Math.PI) / 3)
  )


  const pointLight = new THREE.PointLight(0xffffff,10) // 点光源  
  sphere.add(pointLight)

  pointLightGroup.add(sphere)
}
scene.add(pointLightGroup)

// --------------------------使用补间函数 从0-2pai--------------------------------
const options = {
  angle : 0
}
gsap.to(options,{
  angle:Math.PI * 2,
  duration: 10, // 周期 10秒中转一圈
  ease: "linear",
  repeat: -1, // 重复
  yoyo: true,
  onUpdate: () => {
    pointLightGroup.rotation.y = options.angle // 整个组旋转
    pointLightArr.forEach((item,index) => {
      item.position.set(
        1,
        Math.cos((index *2 * Math.PI ) / 3 + options.angle * 5 ),
        1
      )
    })
  }
})
// --------------------------------------------------------------------------------------------------------------------
// 渲染函数 
const render = function () {
  requestAnimationFrame(render)
  controls.update()
  renderer.render(scene, camera)
}
render()
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
canvas {
  width: 100%;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
}
</style>
