<template>
  <div class="canvas" ref="screenDom"></div>
</template>

<script setup>
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js"; // 加载器
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js"; // 解码器
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader" // 导入环境贴图所用的包
import { Reflector } from "three/examples/jsm/objects/Reflector"; // 反射库
import { ref , onU ,onMounted ,onUnmounted} from 'vue';
let screenDom = ref(null)
onMounted(() => {
  // 创建场景
  let scene = new THREE.Scene();
  // 创建相机
  let camera = new THREE.PerspectiveCamera(
    75,
    screenDom.value.clientWidth / screenDom.value.clientHeight,
    0.1,
    1000
  );
  camera.position.set(0, 1.5, 8);
  // 创建渲染器
  let renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(screenDom.value.clientWidth, screenDom.value.clientHeight);
  screenDom.value.appendChild(renderer.domElement);
  // 创建辅助坐标轴
  // let axes = new THREE.AxesHelper(5);
  // scene.add(axes);
  // 添加控制器
  const control = new OrbitControls(camera, renderer.domElement);
// 自动旋转
  // control.enableDamping = true; // 添加阻尼
  // control.dampingFactor = 0.5; // 阻尼强度
  control.autoRotate = true
  
  // 创建rgbe加载器
  let hdrLoader = new RGBELoader();
  hdrLoader.load("./assets/sky12.hdr", (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    scene.background = texture;
    scene.environment = texture;
  });

  // 添加机器人
  // 设置解压缩的加载器
  let dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath("./draco/gltf/");
  dracoLoader.setDecoderConfig({ type: "js" });
  let gltfLoader = new GLTFLoader();
  gltfLoader.setDRACOLoader(dracoLoader);
  gltfLoader.load("./assets/robot.glb", (gltf) => {
    scene.add(gltf.scene);
  });
  // 添加直线光
  let light1 = new THREE.DirectionalLight(0xffffff, 0.3);
  light1.position.set(0, 10, 10);
  let light2 = new THREE.DirectionalLight(0xffffff, 0.3);
  light1.position.set(0, 10, -10);
  let light3 = new THREE.DirectionalLight(0xffffff, 0.8);
  light1.position.set(10, 10, 10);
  scene.add(light1, light2, light3);

// 添加光阵
  const video = document.createElement("video")
  video.src = "./assets/zp2.mp4"
  video.loop = true
  video.muted = true
  video.play()
  const videoTexture = new THREE.VideoTexture(video) // map

  const videoGeometry = new THREE.PlaneGeometry(16,9)
  const videoMaterial = new THREE.MeshBasicMaterial({
    map:videoTexture,
    transparent:true,
    side:THREE.DoubleSide,
    alphaMap:videoTexture // 透明
  })
  const videoMesh = new THREE.Mesh(videoGeometry,videoMaterial)
  videoMesh.position.set(0 , 0.2 ,0)
  videoMesh.rotation.set( -Math.PI / 2 ,0 , 0 )
  scene.add(videoMesh)


  // 添加镜面反射
  const reflectorGeometry = new THREE.PlaneGeometry(100, 100)
  const reflectorPlane = new Reflector(reflectorGeometry,{
    textureWidth:window.innerWidth,
    textureHeight:window.innerHeight,
    color:0x444444
  })
  reflectorPlane.rotation.x = -Math.PI / 2
  scene.add(reflectorPlane)



function render() {
  control.update()
  requestAnimationFrame(render)
  renderer.render(scene, camera)
}
render()
renderer.render(scene, camera)









  // 监听画面变化，更新渲染画面
  window.addEventListener("resize", () => {
    //   console.log("画面变化了");
    // 更新摄像头
    camera.aspect = window.innerWidth / window.innerHeight;
    //   更新摄像机的投影矩阵
    camera.updateProjectionMatrix();

    //   更新渲染器
    renderer.setSize(window.innerWidth, window.innerHeight);
    //   设置渲染器的像素比
    renderer.setPixelRatio(window.devicePixelRatio);
  });
})

</script>

<style>
* {
  margin: 0;
  padding: 0;
}
.canvas { 
  width: 100vw;
  height: 100vh;
}
</style>