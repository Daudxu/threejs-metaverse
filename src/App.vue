<script setup>
import * as THREE from 'three'
import  Stats  from "three/examples/jsm/libs/stats.module"
import  { Octree }  from "three/examples/jsm/math/Octree.js"
import { Capsule } from "three/examples/jsm/math/Capsule.js"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"
import { OctreeHelper } from "three/examples/jsm/helpers/OctreeHelper.js";
import { onMounted, reactive } from 'vue'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
import * as BufferGeometryUtils from "three/examples/jsm/utils/BufferGeometryUtils.js"
onMounted (()=>{
    const clock = new THREE.Clock();
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x88ccee);
    scene.fog = new THREE.Fog(0x88ccee, 0, 50);

    const camera = new THREE.PerspectiveCamera(
      70,
      window.innerWidth / window.innerHeight,
      0.001,
      1000
    )
    camera.position.set(0, 5, 10)
    
    const backCamera = new THREE.PerspectiveCamera(
      70,
      window.innerWidth / window.innerHeight,
      0.001,
      1000
    );
    backCamera.position.set(0, 5, -10);

    const container = document.getElementById("container");
    const renderer = new THREE.WebGL1Renderer({ antialias: true })
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.VSMShadowMap;
    renderer.outputEncoding = THREE.sRGBEncoding;
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    container.appendChild(renderer.domElement)
  
    const stats = new Stats();
    stats.domElement.style.position = "absolute";
    stats.domElement.style.top = '0px';
    container.appendChild(stats.domElement);
    console.log(stats)
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.target.set(0, 0, 0);
    let activeCamera = camera
    function animate() {
      let delta = clock.getDelta();
      stats.update();
      controls.update();
      renderer.render(scene, activeCamera);
   
      requestAnimationFrame(animate);
    }

    // 创建一个平面
    const planeGeometry = new THREE.PlaneGeometry(20, 20, 1, 1);
    const planMaterial = new THREE.MeshBasicMaterial({
      color: 0xffffff,
      side: THREE.DoubleSide
    });
    const plane = new THREE.Mesh(planeGeometry, planMaterial);
    plane.receiveShadow = true;
    plane.rotation.x = -Math.PI / 2;
    scene.add(plane)

    // const geometry = new THREE.TorusKnotBufferGeometry(1, 0.3, 100, 16);
    const material = new THREE.MeshBasicMaterial({ color: 0x22ff00 });
    let geometryes = [];
    for(let i = 0; i<1000; i++){
      const geometry = new THREE.TorusKnotBufferGeometry(
        0.5 + Math.random() * 0.5,
        0.3,
        50+ parseInt(Math.random() * 50),
        16
      )
      const matrix = new THREE.Matrix4();
      matrix.makeRotationX(Math.random() * Math.PI);
      matrix.makeRotationY(Math.random() * Math.PI);
      matrix.makeRotationZ(Math.random() * Math.PI);
      matrix.makeScale(
        Math.random() * 0.5 + 0.5,
        Math.random() * 0.5 + 0.5,
        Math.random() * 0.5 + 0.5
      )
      matrix.makeTranslation(
        Math.random() * 100 - 50,
        Math.random() * 100 - 50,
        Math.random() * 100 - 50
      )
      geometry.applyMatrix4(matrix)
      geometryes.push(geometry)
    }
    const mergeGeometry = BufferGeometryUtils.mergeBufferGeometries(geometryes)
    let mesh = new THREE.Mesh(mergeGeometry, material)
    scene.add(mesh)
    animate();
});


</script>

<template>
  <div id="container">
  </div>
</template>

<style scoped>
/* .logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
} */
</style>
