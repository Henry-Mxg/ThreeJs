<template>
<div class="home">
  <div class="canvas-container" ref="canvasDom"></div>
  <div class="home-content">
      <div class="home-content-title">
        <h1>汽车展示与选配</h1>
      </div>
      <h2>选择车身颜色</h2>
      <div class="select">
        <div
          class="select-item"
          v-for="(item, index) in colors"
          :key="index"
          @click="selectColor(index)"
        >
          <div
            class="select-item-color"
            :style="{ backgroundColor: item }"
          ></div>
        </div>
      </div>

      <h2>选择贴膜材质</h2>
      <div class="select">
        <div
          class="select-item"
          v-for="(item, index) in materials"
          :key="index"
          @click="selectMaterial(index)"
        >
          <div class="select-item-text">{{ item.name }}</div>
        </div>
      </div>
    </div>
</div>
</template>

<script setup>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
import { onMounted ,ref , reactive } from 'vue'
let canvasDom = ref(null)

// 创建场景
const scene = new THREE.Scene();
// 创建相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 2, 6);
// 创建渲染器
const renderer = new THREE.WebGLRenderer({
  // 抗锯齿
  antialias: true,
});
renderer.setSize(window.innerWidth, window.innerHeight);

const render = () => {
  renderer.render(scene, camera);
  // controls && controls.update();
  requestAnimationFrame(render);
};

let wheel = []
let carBody,frontCar,hoodCar,glassCar
// 创建车身材质
const bodyMaterial = new THREE.MeshPhysicalMaterial({
  color:0xff0000,
  metalness:1, // 金属度 
  roughness:0.5, // 粗糙度
  clearcoat:1, // 清漆
  clearcoatRoughness: 0,    // 清漆�

})
// 创建前脸材质
const frontMaterial = new THREE.MeshPhysicalMaterial({
  color:0xff0000,
  metalness:1, // 金属度 
  roughness:0.5, // 粗糙度
  clearcoat:1, // 清漆
  clearcoatRoughness: 0,    // 清漆�

})
// 创建引擎盖材质
const hoodMaterial = new THREE.MeshPhysicalMaterial({
  color:0xff0000,
  metalness:1, // 金属度 
  roughness:0.5, // 粗糙度
  clearcoat:1, // 清漆
  clearcoatRoughness: 0,    // 清漆�

})
// 创建轮毂材质
const wheelMaterial = new THREE.MeshPhysicalMaterial({
  color:0xff0000,
  metalness:1, // 金属度 
  roughness:0.1, // 粗糙度 

})
// 创建玻璃材质
const glassMaterial = new THREE.MeshPhysicalMaterial({
  color:0xffffff,
  metalness:0, // 金属度 
  roughness:0.01, // 粗糙度 
  transmission:1, // 透明度
  transparent:true

})

//颜色的挑选 
let colors = ["red", "blue", "green", "gray", "orange", "purple"];
const selectColor = (index) => {
  bodyMaterial.color.set(colors[index]);
  frontMaterial.color.set(colors[index]);
  hoodMaterial.color.set(colors[index]);
  wheelMaterial.color.set(colors[index]);
}
const materials = [
  { name:'磨砂',value: 1 }, 
  { name:'冰晶', value:0 }
]
// 材质的挑选
const selectMaterial = (index) => {
  bodyMaterial.clearcoatRoughness = materials[index].value
  frontMaterial.clearcoatRoughness = materials[index].value
  hoodMaterial.clearcoatRoughness = materials[index].value
}
onMounted(() => {
 // 把渲染器插入到dom中
  // console.log(canvasDom.value);
  canvasDom.value.appendChild(renderer.domElement);
  // 初始化渲染器，渲染背景
  renderer.setClearColor("#000");
  scene.background = new THREE.Color("#ccc");
  scene.environment = new THREE.Color("#ccc");
  render();

  // 添加网格地面
  const gridHelper = new THREE.GridHelper(10, 10);
  gridHelper.material.opacity = 0.2;
  gridHelper.material.transparent = true;
  scene.add(gridHelper);


  // 添加控制器
  const controls = new OrbitControls(camera, renderer.domElement)
  controls.update()

  // // 添加gltf汽车模型
  const loader = new GLTFLoader();
  const dracoLoader = new DRACOLoader(); // 解码器
  dracoLoader.setDecoderPath("./draco/gltf/");
  loader.setDRACOLoader(dracoLoader); // 把解码器放在里面
  loader.load("./model/bmw01.glb", (gltf) => {
    const bmw = gltf.scene;
    // console.log(gltf);
    scene.add(bmw);
    bmw.traverse((child) => {
      if(child.isMesh) {
        console.log(child.name);
      }
      // 判断是否是轮毂
      if(child.isMesh && child.name.includes('轮毂')) {
        wheel.push(child );
        child.material = wheelMaterial
      }
      // 判断是否是车身
      if(child.isMesh && child.name.includes('Mesh002')) {
       carBody = child
       carBody.material = bodyMaterial
      }
      // 判断是否是前脸 
      if(child.isMesh && child.name.includes('前脸')) {
       frontCar = child
       child.material = frontMaterial
      }
      // 判断是否是引擎盖
      if(child.isMesh && child.name.includes('引擎盖_1')) {
       hoodCar = child
       child.material = hoodMaterial
      }
      //是否是挡风玻璃
      if(child.isMesh && child.name.includes('挡风玻璃')) {
       glassCar = child
       child.material = glassMaterial
      }
    })
  });



  // 添加灯光
  const light1 = new THREE.DirectionalLight(0xffffff,1);
  light1.position.set(0, 0, 10)
  scene.add(light1);

  // 添加灯光
  const light2 = new THREE.DirectionalLight(0xffffff, 1);
  light2.position.set(0, 0, -10)
  scene.add(light2);
  // 添加灯光
  const light3 = new THREE.DirectionalLight(0xffffff, 1);
  light3.position.set(10, 0, 0)
  scene.add(light3);
  // 添加灯光
  const light4 = new THREE.DirectionalLight(0xffffff, 1);
  light4.position.set(-10, 0, 0)
  scene.add(light4);
  // 添加灯光
  const light5 = new THREE.DirectionalLight(0xffffff, 0.3);
  light5.position.set(0, 10, 0)
  scene.add(light5);
  // 添加灯光
  const light6 = new THREE.DirectionalLight(0xffffff, 0.3);
  light6.position.set(0, -10, 0)
  scene.add(light6)
  // 添加灯光
  const light7 = new THREE.DirectionalLight(0xffffff, 0.3)
  light7.position.set(0, 10, 5)
  scene.add(light7)
  // 添加灯光
  const light8 = new THREE.DirectionalLight(0xffffff, 0.3)
  light8.position.set(0, 10, -5)
  scene.add(light8)
  // 添加灯光
  const light9 = new THREE.DirectionalLight(0xffffff, 0.3)
  light9.position.set(-5, 10, 0)
  scene.add(light9)

  // 监听窗口的变化 
window.addEventListener("resize", () => {
    // 重新设置画布大小
    renderer.setSize(window.innerWidth, window.innerHeight)
    // 重新设置相机的大小
    camera.aspect = window.innerWidth / window.innerHeight
    // 重新设置相机的投影矩阵
    camera.updateProjectionMatrix()
})

})
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
.home-content {
  position: fixed;
  top: 0;
  right: 20px;
}

.select-item-color {
  width: 50px;
  height: 50px;
  border: 1px solid #ccc;
  margin: 10px;
  display: inline-block;
  cursor: pointer;
  border-radius: 10px;
}
.select {
  display: flex;
}
</style>
