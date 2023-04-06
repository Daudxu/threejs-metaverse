<!--  -->
<template>
  <div id="three" ref="three">
    <div ref="le" @mousedown="ddmousedown"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted } from "vue";
import * as dat from "dat.gui";
import {
  ACESFilmicToneMapping,
  AmbientLight,
  AnimationAction,
  AnimationMixer,
  AnimationUtils,
  AxesHelper,
  BackSide,
  CameraHelper,
  Clock,
  Color,
  DirectionalLight,
  DirectionalLightHelper,
  Fog,
  Group,
  HemisphereLight,
  IcosahedronGeometry,
  Mesh,
  MeshBasicMaterial,
  MeshLambertMaterial,
  PerspectiveCamera,
  PlaneGeometry,
  PointLight,
  Raycaster,
  Scene,
  Sphere,
  SpotLight,
  sRGBEncoding,
  TextureLoader,
  Vector3,
  VideoTexture,
  VSMShadowMap,
  WebGLRenderer,
} from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import Stats from 'three/examples/jsm/libs/stats.module.js';

import { Octree } from 'three/examples/jsm/math/Octree.js';
import { OctreeHelper } from 'three/examples/jsm/helpers/OctreeHelper.js';

import { Capsule } from 'three/examples/jsm/math/Capsule.js';


/**
 * 配置
 */
// MARK: 配置=======
const le = ref();
const three = ref();
const donuts = ref();

const scene = new Scene(); // 场景对象Scene
const axes = new AxesHelper(50); // 轴长度

const th = reactive({
  ctrl: new dat.GUI({ width: 200 }),
  renderer: new WebGLRenderer({
    antialias: true,
  }), // 渲染器对象
  ambienLight: new AmbientLight(0xffffff, 0.1), // 自然光
  planeGeometry: new PlaneGeometry(100, 100), // 地面
  spotLight: new SpotLight(0xffffff), // 聚光灯
  pointlight: new PointLight(0xffffff, 6, 60), // 点光源
  directionalLight: new DirectionalLight(0xffffff, 0.2), // 平行光源
  hemisphereLight: new HemisphereLight(0xffffff, 0x00ff00, 1), // 半球光光源
});

scene.background = new Color(0x88ccee);
// scene.fog = new Fog( 0x88ccee, 0, 50 );
// scene.add(axes); // 添加轴


const clock =ref<any>(new Clock());
const stats = ref<any>(Stats());
let helper:any = null;

const GRAVITY = ref<number>(30);
const NUM_SPHERES = ref<number>(100);
const SPHERE_RADIUS = ref<number>(0.2);
const STEPS_PER_FRAME = ref<number>(5);

const worldOctree = new Octree();

const playerCollider = new Capsule(
   new Vector3(0,0.01,0), 
   new Vector3(0,0.01,0),
    0.01 );

const playerVelocity = new Vector3();
const playerDirection = new Vector3();

let playerOnFloor = false;
let mouseTime = 0;

const keyStates:any = {};

/**
 * 影相机
 */
// MARK: 影相机=======
let camera = new PerspectiveCamera(75,window.innerWidth / window.innerHeight,0.1,1000); //透视相机
camera.rotation.order = 'YXZ';

/**
 * 灯光
 */
// MARK: 灯光=======

scene.add(th.ambienLight); // 自然光


th.directionalLight.position.set( - 5, 25, - 1 );
th.directionalLight.castShadow = true;
th.directionalLight.shadow.camera.near = 0.01;
th.directionalLight.shadow.camera.far = 500;
th.directionalLight.shadow.camera.right = 30;
th.directionalLight.shadow.camera.left = - 30;
th.directionalLight.shadow.camera.top    = 30;
th.directionalLight.shadow.camera.bottom = - 30;
th.directionalLight.shadow.mapSize.width = 1024;
th.directionalLight.shadow.mapSize.height = 1024;
th.directionalLight.shadow.radius = 4;
th.directionalLight.shadow.bias = - 0.00006;
scene.add( th.directionalLight );

// const helper = new DirectionalLightHelper(th.directionalLight, 5);
// scene.add(helper);

const dirCameraHelper = new CameraHelper(th.directionalLight.shadow.camera);
// scene.add(dirCameraHelper);

const hemisphereLight = new HemisphereLight( 0x4488bb, 0x002244, 0.5 );
hemisphereLight.position.set( 2, 1, 1 );
scene.add( hemisphereLight );

/**
 * 渲染 场景
 */
// MARK: 渲染 场景=======
th.renderer.shadowMap.enabled = true;
th.renderer.setPixelRatio( window.devicePixelRatio );
th.renderer.setSize( window.innerWidth, window.innerHeight );
th.renderer.shadowMap.enabled = true;
th.renderer.shadowMap.type = VSMShadowMap;
th.renderer.outputEncoding = sRGBEncoding;
th.renderer.toneMapping = ACESFilmicToneMapping;


/**
 * Stats 性能检测
 */
// MARK: 性能检测=======
stats.value.domElement.style.position = 'absolute';
stats.value.domElement.style.top = '0px';


/**
 * 地面
 */
// MARK: 地面=======

/**
 * OrbitControls 轨道控制器
 */
// MARK: OrbitControls 轨道控制器=======

/**
 * 导入模型
 */
// MARK: 导入模型=======

let playerMesh:any= null;
let playerMixer= ref<AnimationMixer>();
let actionWalk= ref<AnimationAction>();
let actionIdle= ref<AnimationAction>();


const lookTarget = new Vector3(0, 2, 0);
const loader = new GLTFLoader();
loader.load("./models/RobotExpressive.glb", gltf => {
  // 角色
  playerMesh = gltf.scene;
  playerMesh.scale.set(0.01, 0.01, 0.01);
  gltf.scene.position.set(0,0,0);
  gltf.scene.rotateY(Math.PI);

  gltf.scene.traverse((child)=>{
    child.receiveShadow = true;
    child.castShadow = true;
  })

  camera.position.set(0, 2.5, -5);
  // camera.lookAt(playMesh.value.position);
  camera.lookAt(lookTarget);

  const pointLight: PointLight = new PointLight(0xffffff, 1);
  pointLight.position.set(0, 2, -1);


  playerMixer.value = new AnimationMixer(gltf.scene);
  console.log(gltf.animations)
  const clipWalk = AnimationUtils.subclip(gltf.animations[10], 'Walking', 0, 30);
  actionWalk.value = playerMixer.value.clipAction(clipWalk);

  const clipIdle = AnimationUtils.subclip(gltf.animations[2], 'Idle', 31, 281);
  actionIdle.value = playerMixer.value.clipAction(clipIdle);
  actionIdle.value.play();

  gltf.scene.add(pointLight);
  gltf.scene.add(camera);
  scene.add(gltf.scene);
});
const loadera = new GLTFLoader();
loadera.load("./models/collision-world.glb", (gltf) => {
  // console.log(gltf);
  scene.add(gltf.scene);
  donuts.value = gltf.scene;

  worldOctree.fromGraphNode( gltf.scene );

  gltf.scene.traverse( (child:any) => {

    if ( child.isMesh ) {
      child.castShadow = true;
      child.receiveShadow = true;
      if ( child.material.map ) {
        child.material.map.anisotropy = 4;
      }
    }
  } );

  helper = new OctreeHelper( worldOctree );
  helper.visible = false;
  scene.add( helper );
  animate();
});

/**
 * 建盘控制人物
 */
// MARK: 建盘控制人物=======
let isWalk = ref<boolean>(false);
const playerHalfHeight = new Vector3(0,0.3, 0);
window.addEventListener("keydown", event => {
  keyStates[event.code] = true;
  if (event.key == "w") {
    if (!isWalk.value) {
      if( actionIdle.value && actionWalk.value){
        crossPlay(actionIdle.value,actionWalk.value);
      }
      isWalk.value = true;
    }
  }
});

window.addEventListener("keyup", event => {
  keyStates[ event.code ] = false;

  if (event.key == "w") {
    if( actionIdle.value && actionWalk.value){
      crossPlay(actionWalk.value,actionIdle.value);
    }
    isWalk.value = false;
  }
});

let ddmousedown =()=>{
  document.body.requestPointerLock();
  mouseTime = performance.now();
}

document.addEventListener( 'mouseup', () => {
  // if ( document.pointerLockElement !== null ) throwBall();
});


let preClientX:number;
window.addEventListener("mousemove", event => {
  if(document.pointerLockElement === document.body){
    playerMesh.rotation.y -= -(event.movementX / 300);
    // playerMesh.rotation.x -= -(event.movementY / 2000);
  }
});


let playerCollisions = ()=> {
  const result = worldOctree.capsuleIntersect( playerCollider );
  playerOnFloor = false;
  if ( result ) {
    playerOnFloor = result.normal.y > 0;
    if ( ! playerOnFloor ) {
      playerVelocity.addScaledVector( result.normal, - result.normal.dot( playerVelocity ) );
    }
    playerCollider.translate( result.normal.multiplyScalar( result.depth ) );
  }

}

let updatePlayer = ( deltaTime:number )=> {

  let damping = Math.exp(-4*deltaTime)-1;
  if ( ! playerOnFloor ) {
    playerVelocity.y-=GRAVITY.value*deltaTime;
    // small air resistance
    damping *= 0.1;
  }

  playerVelocity.addScaledVector( playerVelocity, damping );
  const deltaPosition = playerVelocity.clone().multiplyScalar( deltaTime );
  
  playerCollider.translate( deltaPosition );
  playerCollisions();
  if(playerMesh) playerMesh.position.copy( playerCollider.end );
}

let getForwardVector = ()=>{

  playerMesh.getWorldDirection( playerDirection );
  playerDirection.y = 0;
  playerDirection.normalize();

  return playerDirection;
}

let getSideVector = ()=>{

  playerMesh.getWorldDirection( playerDirection );
  playerDirection.y = 0;
  playerDirection.normalize();
  playerDirection.cross( playerMesh.up );

  return playerDirection;

}

let controls=( deltaTime:number )=>{

  // gives a bit of air control
  const speedDelta = deltaTime * ( playerOnFloor ? 25 : 8 );

  if ( keyStates[ 'KeyW' ] ) {
    playerVelocity.add( getForwardVector().multiplyScalar(speedDelta*0.6));
  }

  if ( keyStates[ 'KeyS' ] ) {
    // playerVelocity.add( getForwardVector().multiplyScalar(-speedDelta));
  }

  if ( keyStates[ 'KeyA' ] ) {
    // playerVelocity.add( getSideVector().multiplyScalar(-speedDelta));
  }

  if ( keyStates[ 'KeyD' ] ) {
    // playerVelocity.add( getSideVector().multiplyScalar(speedDelta));
  }

  if ( playerOnFloor ) {
    if ( keyStates[ 'Space' ] ) {
      playerVelocity.y = 15;
    }
  }

}

let teleportPlayerIfOob=()=>{
  if(!playerMesh) return;
  if ( playerMesh.position.y <= - 25 ) {
    playerCollider.start.set( 0, 0.35, 0 );
    playerCollider.end.set( 0, 1, 0 );
    playerCollider.radius = 0.35;
    playerMesh.position.copy( playerCollider.end );
    playerMesh.rotation.set( 0, 0, 0 );
  }
}


let crossPlay = (curAction:AnimationAction, newAction:AnimationAction)=>{
  curAction.fadeOut(0.3);
  newAction.reset();
  newAction.setEffectiveWeight(1);
  newAction.play();
  newAction.fadeIn(0.3);
}

let render = () => {
  th.renderer.render(scene, camera);
};

//============================================场景搭建end==================================

onMounted(() => {
  le.value.appendChild(th.renderer.domElement);
  le.value.appendChild(stats.value.domElement);
  window.addEventListener(
    "resize",
    () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      th.renderer.setSize(window.innerWidth, window.innerHeight);
      render();
    },
    false
  );
});

//============================================构建场景 start==================================
// MARK: 构建场景=======




//============================================构建场景 end==================================
//============================================gui start==================================

/**
 * dat.gui 插件
 */
// MARK: dat.gui 插件=======

let ctrlObj2 = {};

let basicfolder = th.ctrl.addFolder("MeshBasicMaterial");

basicfolder.add( { debug: false }, 'debug' ).onChange(value=>{
  helper.visible = value;
});

basicfolder.open();

//============================================gui end==================================


let animate=()=> {

  const deltaTime = Math.min(0.05, clock.value.getDelta())/STEPS_PER_FRAME.value;

  //我们在子步骤中寻找碰撞，以降低风险
  //一个物体快速穿过另一个物体而无法检测。

  for ( let i = 0; i < STEPS_PER_FRAME.value; i ++ ) {
    controls( deltaTime );
    updatePlayer( deltaTime );
    teleportPlayerIfOob();
  }

  th.renderer.render( scene, camera );
  stats.value.update();
  if (playerMixer.value) {
    playerMixer.value.update(0.045);
  }
  requestAnimationFrame( animate );
}


onUnmounted(() => {
  th.ctrl.destroy(); // 销毁dat.GUI
});
</script>


