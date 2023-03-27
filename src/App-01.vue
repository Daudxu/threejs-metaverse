<script setup>
import * as THREE from 'three'
import  Stats  from "three/examples/jsm/libs/stats.module"
import  { Octree }  from "three/examples/jsm/math/Octree.js"
import { Capsule } from "three/examples/jsm/math/Capsule.js"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"

import { onMounted, reactive } from 'vue'

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
    
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.target.set(0, 0, 0);

    function animate() {
      let delta = clock.getDelta();
      stats.update();
      controls.update();
      renderer.render(scene, camera);
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
    // 创建一个octree
    const worldOctree = new Octree();
    // 创建一个人的碰撞体
    const playerCollider = new Capsule(
      new THREE.Vector3(0, 0.35, 0),
      new THREE.Vector3(0, 1.35, 0),
      0.35
    );
    console.log('worldOctree', worldOctree)
    console.log('playerCollider', playerCollider)
    // 创建胶囊几何体
    const capsuleGeometry  = new THREE.CapsuleGeometry(0.35, 1, 32);
    const capsuleMaterial = new THREE.MeshBasicMaterial({
      color: 0xff0000,
      side: THREE.DoubleSide
    });
    const capsule = new THREE.Mesh(capsuleGeometry, capsuleMaterial)
    capsule.position.set(0, 0.85, 0);
    capsule.castShadow = true;
    scene.add(capsule)

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
