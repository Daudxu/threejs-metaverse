<template>
  <canvas id="container"></canvas>
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

let scene, camera, renderer, leftPress, cube, mouse, raycaster;

onMounted(() => {
        // 初始化场景
        scene = new THREE.Scene();
        scene.background = new THREE.Color(0xffffff);
        // 创建渲染器
        renderer = new THREE.WebGLRenderer({
            canvas: document.getElementById("container"),
            antialias: true, // 抗锯齿
            alpha: true
        });
        renderer.setSize(window.innerWidth, window.innerHeight);


        // 创建透视相机
        camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 40, 30);
        camera.lookAt(0, 0, 0);

        // 参数初始化
        mouse = new THREE.Vector2();
        raycaster = new THREE.Raycaster();

        // 环境光
        var ambientLight = new THREE.AmbientLight(0x606060);
        scene.add(ambientLight);
        // 平行光
        var directionalLight = new THREE.DirectionalLight(0xBCD2EE);
        directionalLight.position.set(1, 0.75, 0.5).normalize();
        scene.add(directionalLight);

        var grid = new THREE.GridHelper(100, 20, 0xFF0000, 0x000000);
        grid.material.opacity = 0.1;
        grid.material.transparent = true;
        scene.add(grid);

        var axesHelper = new THREE.AxesHelper(30);
        scene.add(axesHelper);
        var geometry = new THREE.BoxGeometry(5, 5, 5);
        var material = new THREE.MeshPhongMaterial({color: 0x00ff00});
        cube = new THREE.Mesh(geometry, material);
        scene.add(cube);
        function animate(){
          requestAnimationFrame(animate);
           renderer.render(scene, camera);
        }
        animate();

  window.addEventListener(
    "mousemove",
    (event) => {
      event.preventDefault();
        if (leftPress) {
          console.log(event)
            cube.rotateOnAxis(
                new THREE.Vector3(0, 1, 0),
                event.movementX / 500
            );
            cube.rotateOnAxis(
                new THREE.Vector3(1, 0, 0),
                event.movementY / 500
            );
        }
    },
    false
  )

  window.addEventListener(
    "mousedown",
    (event) => {
      event.preventDefault();
      leftPress = true;
    },
    false
  )

  window.addEventListener(
    "mouseup",
    (event) => {
      event.preventDefault();
      leftPress = false;
    },
    false
  )

});





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
</style>
