<script setup>
// 引入组合式API
import { ref, onMounted } from 'vue'

// 引入threejs
import * as THREE from 'three'

// 引入动画库 gsap
import gsap from 'gsap'

// 引入轨道控制器
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

// 获取画布节点
const sceneDom = ref(null)

// 经纬度坐标转球面坐标计算公式
function lon2xyz (R, longitude, latitude) {
  // 转弧度值
  let lon = (longitude * Math.PI) / 180
  let lat = (latitude * Math.PI) / 180
  // js坐标系z坐标轴对应经度-90°，而不是90°
  lon = -lon

  // 经纬度坐标转球面坐标计算公式
  const x = R * Math.cos(lat) * Math.cos(lon);
  const y = R * Math.sin(lat);
  const z = R * Math.cos(lat) * Math.sin(lon);
  // 返回球面坐标
  return new THREE.Vector3(x, y, z)
}

// 组件挂载完毕后
onMounted(() => {
  // 创建场景
  const scene = new THREE.Scene()

  const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    100000
  )
  camera.position.set(0, 50, 300)
  scene.add(camera)

  // 创建渲染器
  const renderer = new THREE.WebGLRenderer({antialias: true})
  renderer.setSize(window.innerWidth, window.innerHeight)

  sceneDom.value.appendChild( renderer.domElement )

  // 创建控制器
  const controls = new OrbitControls(camera, sceneDom.value)
  controls.enableDamping = true
  controls.autoRotate = true
  controls.update()

  // 场景背景
  scene.background = new THREE.Color(0x030311)

  // 创建星空背景
  const vertices = []
  for (let i = 0; i < 500; i++) {
    const vertex = new THREE.Vector3()
    vertex.x = 800 * Math.random() - 400
    vertex.y = 800 * Math.random() - 400
    vertex.z = 800 * Math.random() - 400
    vertices.push(vertex.x, vertex.y, vertex.z)
  }

  // 星空效果
  const starGeometry = new THREE.BufferGeometry()
  starGeometry.setAttribute('position',
  new THREE.BufferAttribute(new Float32Array(vertices), 3))

  // 加载点材质
  const starTexture = new THREE.TextureLoader().load('/images/stars.png')
  const starMaterial = new THREE.PointsMaterial({
    size: 2,
    color: 0x4d76cf,
    // 指定点的大小是否因相机深度而衰减。（仅限透视摄像头。）默认为true。
    sizeAttenuation: true,
    map: starTexture,
    opacity: 1,
    transparent: true
  })

  // 创建漫天星
  const stars = new THREE.Points(starGeometry, starMaterial)
  scene.add(stars)

  // 渲染函数
  function animate () {
    requestAnimationFrame( animate )
    renderer.render( scene, camera )
  }

  // 创建地球
  const earthGeometry = new THREE.SphereGeometry(50, 32, 32)
  const earthTexture = new THREE.TextureLoader().load('/images/map.jpg')
  const earthMaterial = new THREE.MeshBasicMaterial({
    map: earthTexture
  })

  const earth = new THREE.Mesh(earthGeometry, earthMaterial)
  scene.add(earth)

  // 发光的地球
  const lightEarthTexture = new THREE.TextureLoader().load('/images/earth.jpg')
  const lightEarthMaterial = new THREE.MeshBasicMaterial({
    map: lightEarthTexture,
    alphaMap: lightEarthTexture,
    // 在使用此材质显示对象时要使用何种混合。
    blending: THREE.AdditiveBlending,
    transparent: true
  })

  const lightEarth = new THREE.Mesh(earthGeometry, lightEarthMaterial)
  scene.add(lightEarth)

  // 创建地球内 外发光点精灵材质
  const spriteTexture = new THREE.TextureLoader().load('/images/glow.png')
  const spriteMaterial = new THREE.SpriteMaterial({
    map: spriteTexture,
    color: 0x4d76cf,
    transparent: true,
    // 渲染此材质是否对深度缓冲区有任何影响。默认为true。
    depthWrite: false,
    // 是否在渲染此材质时启用深度测试。默认为 true。
    depthTest: false,
    blending: THREE.AdditiveBlending
  })

  const sprite = new THREE.Sprite(spriteMaterial)
  sprite.scale.set(155, 155, 0)
  scene.add(sprite)

  // 创建地球内 内发光点精灵
  const spriteTexture1 = new THREE.TextureLoader().load('/images/innerGlow.png')
  const spriteMaterial1 = new THREE.SpriteMaterial({
    map: spriteTexture1,
    color: 0x4d76cf,
    transparent: true,
    // 渲染此材质是否对深度缓冲区有任何影响。默认为true。
    depthWrite: false,
    // 是否在渲染此材质时启用深度测试。默认为 true。
    depthTest: false,
    blending: THREE.AdditiveBlending
  })

  const sprite1 = new THREE.Sprite(spriteMaterial1)
  sprite.scale.set(128, 128, 0)
  scene.add(sprite1)

  const scale = new THREE.Vector3(1, 1, 1)

  // 实现光柱
  for (let i = 0; i < 30; i++) {
    const lightPillarTexture = new THREE.TextureLoader().load('/images/light_column.png')
    const lightPillarGeometry = new THREE.PlaneGeometry(3, 20)
    const lightPillarMaterial = new THREE.MeshBasicMaterial({
      color: 0xffffff,
      map: lightPillarTexture,
      alphaMap: lightPillarTexture,
      transparent: true,
      blending: THREE.AdditiveBlending,
      side: THREE.DoubleSide,
      depthWrite: false
    })

    const lightPillar = new THREE.Mesh(lightPillarGeometry, lightPillarMaterial)
    lightPillar.add(lightPillar.clone().rotateY(Math.PI / 2))

    // 创建波纹扩散效果
    const circlePlane = new THREE.PlaneGeometry(6, 6)
    const circleTexture = new THREE.TextureLoader().load('/images/label.png')
    const circleMaterial = new THREE.MeshBasicMaterial({
      map: circleTexture,
      color: 0xffffff,
      transparent: true,
      blending: THREE.AdditiveBlending,
      depthWrite: false,
      side: THREE.DoubleSide
    })

    const circleMesh = new THREE.Mesh(circlePlane, circleMaterial)

    circleMesh.rotation.x = -Math.PI / 2
    circleMesh.position.set(0, -7, 0)
    lightPillar.add(circleMesh)

    // 对波纹作动画
    gsap.to(circleMesh.scale, {
      duration: 1 + Math.random() * 0.5,
      x: 2,
      y: 2,
      z: 2,
      repeat: -1,
      delay: Math.random() * 0.5,
      yoyo: true,
      ease: 'power2.inOut'
    })

    // 光柱位置
    const lat = Math.random() * 180 -90
    const lon = Math.random() * 360 -180
    const position = lon2xyz(60, lon, lat)
    lightPillar.position.set( position.x, position.y, position.z )

    // 表示对象局部旋转的Quaternion（四元数）。
    lightPillar.quaternion.setFromUnitVectors(
      new THREE.Vector3(0, 1, 0),
      position.clone().normalize()
    )
    scene.add(lightPillar)
  }

  // 创建围绕地球运行的月球
  const moonTexture = new THREE.TextureLoader().load('/images/moon.jpg')
  const moonGeometry = new THREE.SphereGeometry( 5, 32, 32)
  const moonMaterial = new THREE.MeshStandardMaterial({
    map: moonTexture,
    emissive: 0xffffff,
    emissiveMap: moonTexture
  })

  const moon = new THREE.Mesh( moonGeometry, moonMaterial )
  moon.position.set(150, 0, 0)
  scene.add(moon)

  // 创建月球环
  const moonRingTexture = new THREE.TextureLoader().load('/images/moon_ring.png')
  const moonRingGeometry = new THREE.RingGeometry(145, 155, 64)
  const moonRingMaterial = new THREE.MeshBasicMaterial({
    map: moonRingTexture,
    transparent: true,
    blending: THREE.AdditiveBlending,
    side: THREE.DoubleSide,
    depthWrite: false,
    opacity: 0.5
  })

  const moonRing = new THREE.Mesh(moonRingGeometry, moonRingMaterial)
  moonRing.rotation.x = -Math.PI / 2
  scene.add(moonRing)

  // 控制月球围绕月球环旋转
  let time = {
    value: 0
  }
  gsap.to(time, {
    value: 1,
    duration: 10,
    repeat: -1,
    ease: "linear",
    onUpdate: () => {
      moon.position.x = 150 * Math.cos(time.value * Math.PI * 2);
      moon.position.z = 150 * Math.sin(time.value * Math.PI * 2);
      moon.rotation.y = time.value * Math.PI * 8;
    },
  })

  animate()
})

</script>

<template>
  <div class="home">
    <div class="canvas-container" ref="sceneDom"></div>
  </div>
</template>

<style scoped>

</style>
