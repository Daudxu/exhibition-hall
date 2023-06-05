<script setup>
import * as THREE from 'three'
import  Stats  from "three/examples/jsm/libs/stats.module"

// import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"
import * as CANNON from "cannon-es"
import { onMounted, reactive } from 'vue'
import CannonDebugger from 'cannon-es-debugger'
import { PointerLockControlsCannon } from './PointerLockControlsCannon'
import nipplejs from 'nipplejs';

onMounted (()=>{
  // 默认场景
  const scene = new THREE.Scene();
  scene.background = new THREE.Color(0x88ccee);
  scene.fog = new THREE.Fog(0x88ccee, 0, 50);
  // 场景1
  const scene2 = new THREE.Scene();
  // 场景2
  const scene3 = new THREE.Scene();

  // 相机
  const camera = new THREE.PerspectiveCamera(
    70,
    window.innerWidth / window.innerHeight,
    0.001,
    1000
  )
  camera.position.set(0, 2, 4)
  camera.lookAt(0, 0, -2)
  // 渲染器
  const container = document.getElementById("container");
  const renderer = new THREE.WebGL1Renderer({ antialias: true })
  renderer.setPixelRatio(window.devicePixelRatio);
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.shadowMap.enabled = true;
  renderer.shadowMap.type = THREE.VSMShadowMap;
  renderer.outputEncoding = THREE.sRGBEncoding;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  container.appendChild(renderer.domElement)

  // 状态信息
  const stats = new Stats();
  stats.domElement.style.position = "absolute";
  stats.domElement.style.top = '0px';
  container.appendChild(stats.domElement);

  // 轨道控制器
  // const controls = new OrbitControls(camera, renderer.domElement);
  // controls.target.set(0, 0, 0);
  
  // 初始化物理世界
  let world = new CANNON.World();
  world.gravity.set(0, -9.82, 0)
  const cannonDebugger = new CannonDebugger(scene, world, {})
  // 默认弹性和摩擦系数
  world.defaultContactMaterial.contactEquationStiffness = 1e9;
  world.defaultContactMaterial.contactEquationRelaxation = 4;
  // 迭代求解器，用于解决约束力学问题
  const solver = new CANNON.GSSolver();
  // 物理引擎在计算运动和碰撞时要执行的迭代步骤数
  solver.iterations = 7;
  // 物理引擎在计算过程中所允许的最大误差
  solver.tolerance = 0.1;

  world.solver = new CANNON.SplitSolver(solver)
  
  // 创建物理材质
  const physicsMaterial = new CANNON.Material('physics');
  const physics_physics = new CANNON.ContactMaterial(
    physicsMaterial,
    physicsMaterial,
    {
      friction: 0,
      restitution: 0.3,
    }
  )
  // 将物理材质添加到世界中
  world.addContactMaterial(physics_physics);

  // 创建一个球体
  const radius = 0.8;
  const sphereShape = new CANNON.Sphere(radius);
  const sphereBody = new CANNON.Body({
    mass: 100,
    material: physicsMaterial,
  })
  sphereBody.addShape(sphereShape);
  sphereBody.linearDamping = 0.9;
  sphereBody.position.set(0, 5, 0)
  world.addBody(sphereBody)
  // 创建一个球体
  const sphereGeometry = new THREE.SphereGeometry(radius, 8, 8);
  const sphereMaterial = new THREE.MeshStandardMaterial({
    color: 0xff0000,
    wireframe: true
  })
  const sphereMesh = new THREE.Mesh(sphereGeometry,sphereMaterial)
  scene.add(sphereMesh)

  // 创建物理地面
  const groundShape = new CANNON.Plane();
  const groundBody = new CANNON.Body({
    mass: 0,
    material: physicsMaterial,
  });
  groundBody.addShape(groundShape);
  groundBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), -Math.PI / 2)
  world.addBody(groundBody)
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
  
  let controls

  function initPointerLock() {
      controls = new PointerLockControlsCannon(camera, sphereBody);
      scene.add(controls.getObject());

      renderer.domElement.addEventListener("click", () => {
          controls.lock()
      })
      controls.addEventListener("lock", () => {
        controls.enabled = true
      })
      controls.addEventListener("unlock", () => {
        controls.enabled = false
      })
  }
  initPointerLock();
  const clock = new THREE.Clock();
  function animate() {
    let delta = clock.getDelta();
    world.step(1 / 60, delta, 10);
    stats.update();
    controls.update(delta);
  
    cannonDebugger.update()
    sphereMesh.position.copy(sphereBody.position)
    sphereMesh.quaternion.copy(controls.getObject().quaternion)
    
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  }


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

    animate();
});


</script>

<template>
  <div id="container">
  </div>
</template>

<style scoped>

</style>
