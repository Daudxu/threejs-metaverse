<script setup>
import * as THREE from 'three'
import  Stats  from "three/examples/jsm/libs/stats.module"
// import  { Octree }  from "three/examples/jsm/math/Octree.js"
// import { Capsule } from "three/examples/jsm/math/Capsule.js"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"
import * as CANNON from "cannon-es"
// import * as CANNON from "cannon"
import { onMounted, reactive } from 'vue'
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
// import { threeToCannon, ShapeType } from 'three-to-cannon';
import { threeToCannon } from './utils/three-to-cannon';
import CannonDebugger from 'cannon-es-debugger'
import * as _ from 'lodash';

onMounted (()=>{

    // 场景初始化
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x88ccee);
    // scene.fog = new THREE.Fog(0x88ccee, 0, 50);
    // 创建相机
    const camera = new THREE.PerspectiveCamera(80, window.innerWidth / window.innerHeight, 0.1, 1010);
    camera.position.set(0, 350, 120)
    // 渲染
    const container = document.getElementById("container");
    const renderer = new THREE.WebGLRenderer({ antialias: true })
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.VSMShadowMap;
    renderer.outputEncoding = THREE.sRGBEncoding;
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    container.appendChild(renderer.domElement)
    // 性能监控
    const stats = new Stats();
    stats.domElement.style.position = "absolute";
    stats.domElement.style.top = '0px';
    container.appendChild(stats.domElement);

    // 轨道控制
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.target.set(0, 0, 0);

    // let dracoLoader = new DRACOLoader()
    // let loader = new  GLTFLoader()
    // dracoLoader.setDecoderPath("./draco/gltf/");
    // dracoLoader.setDecoderConfig({type: "js"});
    // loader.setDRACOLoader(loader);

    // 创建物理世界
    const world = new CANNON.World();
    world.gravity.set(0, -100, 0);

    const cannonDebugger = new CannonDebugger(scene, world, {})

 
    let robotModel = null
    const gltfLoader = new GLTFLoader();
    const dracoLoader = new DRACOLoader();
    dracoLoader.setDecoderPath("./draco/gltf/");
    dracoLoader.setDecoderConfig({ type: "js" });
    dracoLoader.preload();
    gltfLoader.setDRACOLoader(dracoLoader);
    gltfLoader.load('./models/world.glb', gltf => {
      gltf.scene.traverse((child) => {
        if (child.hasOwnProperty('userData')){
            if (child.type === 'Mesh'){
                // Utils.setupMeshProperties(child);
                // this.sky.csm.setupMaterial(child.material);

                // if (child.material.name === 'ocean')
                // {
                //   this.registerUpdatable(new Ocean(child, this));
                // }
            }

            if (child.userData.hasOwnProperty('data')){
                if (child.userData.data === 'physics')
                {
                  if (child.userData.hasOwnProperty('type')) 
                  {
                    // Convex doesn't work! Stick to boxes!
                    if (child.userData.type === 'box'){
                      // let phys = new BoxCollider({size: new THREE.Vector3(child.scale.x, child.scale.y, child.scale.z)});
                      // phys.body.position.copy(Utils.cannonVector(child.position));
                      // phys.body.quaternion.copy(Utils.cannonQuat(child.quaternion));
                      // phys.body.computeAABB();

                      // phys.body.shapes.forEach((shape) => {
                      //   shape.collisionFilterMask = ~CollisionGroups.TrimeshColliders;
                      // });

                      // this.physicsWorld.addBody(phys.body);
                    } else if (child.userData.type === 'trimesh') {
                      let phys = getBody(child, {});
                      // console.log("phys", phys)
                      world.addBody(phys);
                    }

                    child.visible = false;
                  }
                }

                if (child.userData.data === 'path')
                {
                  // this.paths.push(new Path(child));
                }

                if (child.userData.data === 'scenario')
                {
                  // this.scenarios.push(new Scenario(child, this));
                }
            }

        }
      })
      robotModel = gltf.scene
      // gltf.scene.name = config.name
      // gltf.scene.rotation.set(0, 0, 0)
      // resolve(gltf)
      scene.add(gltf.scene)
    })

    const getBody = (mesh, options) => {
  
      const meshClone = mesh.clone();

      let defaults = {
        mass: 0,
        position: mesh.position,
        rotation: mesh.quaternion,
        friction: 0.3
      };
      options = setDefaults(options, defaults);

      let mat = new CANNON.Material('triMat');
      mat.friction = options.friction;
      var Type = {
        BOX: 'Box',
        CYLINDER: 'Cylinder',
        SPHERE: 'Sphere',
        HULL: 'ConvexPolyhedron',
        MESH: 'Trimesh'
      };

      let shape = threeToCannon(meshClone, {type: Type.MESH});

      // Add phys sphere
      let physBox = new CANNON.Body({
        mass: options.mass,
        position: options.position,
        quaternion: options.rotation,
        shape: shape
      });

      physBox.material = mat;
      return physBox
    }

    const setDefaults = (options, defaults) => {
         return _.defaults({}, _.clone(options), defaults);
    }
    // 创建物理小球形状
    const sphereShape = new CANNON.Sphere(0.5);
    // 设置物体材质
    // const sphereWorldMaterial = new CANNON.Material();
    // 创建物理世界的物体
    const sphereBody = new CANNON.Body({
        shape: sphereShape,
        // 位置
        position: new CANNON.Vec3(0, 5, 0),
        // 小球质量
        mass: 1,
        // 物体材质
        // material: sphereWorldMaterial,
    });
    // 将物体添加至物理世界
    world.addBody(sphereBody);
    // 创建地面
    const floorShape = new CANNON.Plane();
    const floorBody = new CANNON.Body();
    // 当质量为0时，使得物体保持不动
    floorBody.mass = 0;
    floorBody.addShape(floorShape);
    // 位置
    floorBody.position.set(0, 0, 0);
    // floorBody.position.x = -Math.PI / 2;
    // 旋转
    floorBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), -Math.PI / 2);
    world.addBody(floorBody);
    // 创建一个平面
    const planeGeometry = new THREE.PlaneGeometry(20, 20, 1, 1);
    const planMaterial = new THREE.MeshBasicMaterial({
      color: 0xffffff,
      side: THREE.DoubleSide
    });
    const plane = new THREE.Mesh(planeGeometry, planMaterial);
    plane.receiveShadow = true;
    plane.rotation.x = -Math.PI / 2;
    // plane.position.set(0, 0, 0);
    // plane.position.copy(floorBody.position)
    scene.add(plane)

    const pointLight1 = new THREE.PointLight(0xffffff,1.0);
    const pointLight2 = new THREE.PointLight(0xffffff,1.0);
    pointLight1.position.set(400, 100, 200);
    pointLight2.position.set(-400, 100, -200);
    scene.add(pointLight1);
    scene.add(pointLight2);
    const light = new THREE.AmbientLight( 0x404040 ); // soft white light
    scene.add( light );
    // let pointLightHelper = new THREE.PointLightHelper( pointLight1, 1 );
    // scene.add( pointLightHelper );
    
    // // 创建胶囊几何体
    // const capsuleGeometry  = new THREE.CapsuleGeometry(0.35, 1, 32);
    // const capsuleMaterial = new THREE.MeshBasicMaterial({
    //   color: 0xff0000,
    //   side: THREE.DoubleSide
    // });
    // const capsule = new THREE.Mesh(capsuleGeometry, capsuleMaterial)
    // capsule.position.set(0, 0.85, 0);
    // capsule.castShadow = true;
    // scene.add(capsule)
    const clock = new THREE.Clock();

    function animate() {
      let delta = clock.getDelta();
      world.step(1 / 60, delta, 10);
      if(robotModel){
        // const result = threeToCannon(robotModel);
        // console.log('robotModel', result)
        // robotModel.position.copy(sphereBody.position)
        sphereBody.position.copy(robotModel.position)
      }
    
      // capsule.quaternion.copy(sphereBody.position)
      stats.update();
      controls.update();
      renderer.render(scene, camera);
      cannonDebugger.update()
      requestAnimationFrame(animate);
    }
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
