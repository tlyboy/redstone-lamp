<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/Addons.js'
import { RGBELoader } from 'three/addons/loaders/RGBELoader.js'

const el = ref<HTMLElement | null>(null)

onMounted(() => {
  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000,
  )

  const renderer = new THREE.WebGLRenderer({ alpha: true })
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.setSize(window.innerWidth, window.innerHeight)
  el.value!.appendChild(renderer.domElement)

  new RGBELoader().load('textures/modern_bathroom_1k.hdr', function (texture) {
    texture.mapping = THREE.EquirectangularReflectionMapping
    scene.background = texture
    scene.environment = texture
  })

  // 使用辅助器
  const controls = new OrbitControls(camera, renderer.domElement)

  controls.autoRotate = true

  let redstoneLampTexture = new THREE.TextureLoader().load(
    'textures/redstone_lamp.png',
  )

  let redstoneLampOnTexture = new THREE.TextureLoader().load(
    'textures/redstone_lamp_on.png',
  )

  const geometry = new THREE.BoxGeometry(1, 1, 1)
  const material = new THREE.MeshBasicMaterial({ map: redstoneLampTexture })
  const cube = new THREE.Mesh(geometry, material)
  scene.add(cube)

  camera.position.x = 3
  camera.position.y = 2
  camera.position.z = 1

  function animate() {
    controls.update()

    requestAnimationFrame(animate)

    renderer.render(scene, camera)
  }

  animate()

  window.addEventListener('resize', () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  })

  window.addEventListener('click', (event: MouseEvent) => {
    let raycaster = new THREE.Raycaster()
    let mouse = new THREE.Vector2()
    mouse.x = (event.clientX / window.innerWidth) * 2 - 1
    mouse.y = -(event.clientY / window.innerHeight) * 2 + 1
    raycaster.setFromCamera(mouse, camera)
    let intersects = raycaster.intersectObjects(scene.children)
    if (intersects.length > 0) {
      if (intersects[0].object === cube) {
        if (cube.material.map === redstoneLampTexture) {
          cube.material.map = redstoneLampOnTexture
          ElMessage.success('灯已开！')
        } else {
          cube.material.map = redstoneLampTexture
          ElMessage.warning('灯已关！')
        }
        cube.material.needsUpdate = true
      }
    }
  })
})
</script>

<template>
  <div ref="el"></div>
</template>
