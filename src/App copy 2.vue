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

import io from 'socket.io-client'
import TWEEN from '@tweenjs/tween.js'
import gsap from "gsap"

const socket = io('http://localhost:3000');

let leftPress;

onMounted(() => {
  // 场景
  const scene = new THREE.Scene();
  // 相机
  const camera = new THREE.PerspectiveCamera(
    70,
    window.innerWidth / window.innerHeight,
    0.001,
    1000
  );
  camera.position.set(0, 5, 10);

  // 模型
  const geometry = new THREE.BoxGeometry( 1, 1, 1 );
  const material = new THREE.MeshBasicMaterial( {color: 0x00ff00} );
  let model =  new THREE.Mesh( geometry, material );
  scene.add( model );
  // 目标位置点
  const targetPos = new THREE.Vector3(10,10,10)   
  // 目标移动时的朝向偏移
  const offsetAngle = Math.PI/2  
  // 创建一个4维矩阵
  var mtx = new THREE.Matrix4()
  // 设置朝向
  mtx.lookAt(model.position.clone() , targetPos , model.up) 
  mtx.multiply(new THREE.Matrix4().makeRotationFromEuler(new THREE.Euler(0 , offsetAngle, 0 )))
  // 计算出需要进行旋转的四元数值
  var toRot = new THREE.Quaternion().setFromRotationMatrix(mtx)
  // 使用Tween线性改变model的position。此处的action方法Tween官方可能没有，你可以使用Tween的其他方法，只要能线性插值改变position就可以了。
  // TWEEN.action(model.position , 1000 , targetPos ,TWEEN.linear,function(){
  //   //oncomplete
  //   },function(){
  //     //onupdate
  //     model.quaternion.slerp(toRot , 0.2)  //应用旋转。0.2代表插值step。可以做到平滑旋转过渡
  //   }
  // )

// 添加动画
gsap.to(
   model.position,
    {
        x: 10,
        duration: 5,
        ease: 'power1.inOut', 
        repeat: -1,
        yoyo: true, 
        onUpdate(){
          model.quaternion.slerp(toRot , 0.2)
        }
    }
)

  model.quaternion.slerp(toRot , 0.2)  

  const container = document.getElementById("container");

  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.VSMShadowMap;
  renderer.outputEncoding = THREE.sRGBEncoding;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  container.appendChild(renderer.domElement);

  const controls = new OrbitControls(camera, renderer.domElement);
  controls.target.set(0, 0, 0);
  function animate() {
    // let delta = clock.getDelta();
    // console.log(delta);
    controls.update();
    // 更新动作
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }

  animate()
})

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
  z-index: 99999;
}
</style>
