<template>
  <div id="container"></div>
  <div class="cl-option">
    <button @click="handleClickAddAvatar"> dd Avatar</button>
  </div>
</template>

<script setup>
import * as THREE from "three";
import Stats from "three/examples/jsm/libs/stats.module.js";

import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

import { Octree } from "three/examples/jsm/math/Octree.js";
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper.js";

import { Capsule } from "three/examples/jsm/math/Capsule.js";

import { GUI } from "three/examples/jsm/libs/lil-gui.module.min.js";

import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";

import { reactive, onMounted, ref } from "vue";
import * as SkeletonUtils from 'three/examples/jsm/utils/SkeletonUtils'
import * as BufferGeometryUtils from "three/examples/jsm/utils/BufferGeometryUtils.js";
import { RGBELoader } from "three/examples/jsm/loaders/RGBELoader";
import io from 'socket.io-client'

const socket = io('http://localhost:3000');
const clock = new THREE.Clock();
const scene = new THREE.Scene();


onMounted(() => {
  scene.background = new THREE.Color(0x88ccee);
  // scene.fog = new THREE.Fog(0x88ccee, 0, 50);

  const camera = new THREE.PerspectiveCamera(
    70,
    window.innerWidth / window.innerHeight,
    0.001,
    1000
  );
  camera.position.set(0, 5, 10);

  const container = document.getElementById("container");

  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.VSMShadowMap;
  renderer.outputEncoding = THREE.sRGBEncoding;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  container.appendChild(renderer.domElement);

  const stats = new Stats();
  stats.domElement.style.position = "absolute";
  stats.domElement.style.top = "0px";
  container.appendChild(stats.domElement);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.target.set(0, 0, 0);

  // console.log(renderer.info);


  function animate() {
    let delta = clock.getDelta();

    stats.update();
    controls.update();
    cubeCamera.update(renderer, scene);
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }

  const hdrLoader = new RGBELoader();
  hdrLoader.load("./hdr/023.hdr", (texture) => {
    texture.mapping = THREE.EquirectangularReflectionMapping;
    texture.format = THREE.RGBAFormat;
    scene.background = texture;
    scene.environment = texture;
    sphereMaterial.envMap = cubeRenderTarget.texture;
  });

  // 创建球
  const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
  const sphereMaterial = new THREE.MeshPhysicalMaterial({
    color: 0xffffff,
    // 透光程度
    transparent: true,
    roughness: 0,
    metalness: 1,
  });
  sphereMaterial._iridescence = 1;
  const sphere = new THREE.Mesh(sphereGeometry, sphereMaterial);
  sphere.position.set(0, 0, 0);
  scene.add(sphere);

  var box1 = createAvatar()
  box1.position.set(Math.random() * 10 - 2, 0, Math.random() * 10 - 2)
  scene.add(box1)

  // 创建 cubeTarget
  const cubeRenderTarget = new THREE.WebGLCubeRenderTarget(512);
  const cubeCamera = new THREE.CubeCamera(0.1, 1000, cubeRenderTarget);
  // 添加角色到场景
  socket.on("connect", () => {
    console.log('ok'); 
    // socket.emit("connect", "world");
    // socket.emit('connect', 'a');
  });
  socket.on("message", (e) => {
    // console.log(e); 
    if(typeof e === 'object') {
      console.log(e); 
      // var m = SkeletonUtils.clone(scene)
      // var  position = new THREE.Vector3(e.x, e.y, e.z)
      const box = createAvatar()
      box.position.set(e.x, e.y, e.z)
      // box.position.set(position)
      scene.add(box)
    }

    // var object3D = new THREE.Object3D();
    //  object3D.add(e)
    // scene.add(object3D)
    // socket.emit("connect", "world");
    // socket.emit('message', 'a');
  });

  animate();
});
const handleClickAddAvatar = () => {
  // console.log("===")
  var avatar = createAvatar()
  avatar.position.set(Math.random() * 10 - 2, Math.random() * 10 - 2, Math.random() * 10 - 2)
  socket.emit("message", avatar.position);
  scene.add(avatar)
}

const createAvatar = () => {
  const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
  const boxMaterial = new THREE.MeshPhysicalMaterial({
    color: 0xffffff
  })
  const box = new THREE.Mesh(boxGeometry, boxMaterial)
  // var object3D = new THREE.Object3D();
  // object3D.add(box)
  return box
}
</script>

<style>
* { 
  margin: 0;
  padding: 0;
}
#container {
  width: 100vw;
  height: 100vh;
}
.cl-option {
  position: fixed;
  right: 0;
  top: 0;
}
</style>
