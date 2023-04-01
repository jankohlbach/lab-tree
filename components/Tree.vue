<script setup lang="ts">
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'
import { Pane } from 'tweakpane'

const canvas = ref()

onMounted(() => {
  const pane = new Pane()

  const scene = new THREE.Scene()

  const axesHelper = new THREE.AxesHelper()
  scene.add(axesHelper)

  const camera = new THREE.PerspectiveCamera(50, window.innerWidth / window.innerHeight, 0.1, 100)
  camera.position.z = 25
  const cameraFolder = pane.addFolder({ title: 'camera' })
  cameraFolder.addInput(camera.position, 'z', { min: 1, max: 100 })

  const axiom = 'F'
  let sentence = ''
  const rule = 'F[---F][---<<<<<<<<F][--->>>>>>>>F]'
  const iterations = 2
  const angle = 15
  let length = 10
  const lengthModifier = 0.7
  let radius = 0.18
  const radialModifier = 0.8
  const color = [100, 85, 75]

  const createOrigin = () => {
    const geometry = new THREE.SphereGeometry(1, 1, 1)
    const material = new THREE.MeshBasicMaterial({ transparent: true, opacity: 0 })
    return new THREE.Mesh(geometry, material)
  }

  const segment = (length: number, radius: number, angleX: number, angleY: number, angleZ: number, color: any[], previousLength: number) => {
    const geometry = new THREE.CylinderGeometry(radius * radialModifier, radius, length, radius * 200, radius * 200)
    geometry.translate(0, length / 2, 0)

    const material = new THREE.MeshPhongMaterial({ color: `rgb(${color[0]}, ${color[1]}, ${color[2]})`, dithering: true, flatShading: true, precision: 'lowp' })
    const cylinder = new THREE.Mesh(geometry, material)

    cylinder.rotation.x = (angleX * Math.PI / 180)
    cylinder.rotation.y = (angleY * Math.PI / 180)
    cylinder.rotation.z = (angleZ * Math.PI / 180)

    cylinder.position.set(0, previousLength, 0)

    return cylinder
  }

  const constructScene = (sentence: string) => {
    let angleX = 0
    let angleY = 0
    let angleZ = 0
    const states = []
    const origin = createOrigin()

    origin.position.set(0, -14, 0)

    scene.add(origin)

    let previousSegment: THREE.Mesh<THREE.SphereGeometry | THREE.CylinderGeometry, THREE.MeshBasicMaterial> = origin
    let previousLength = 0

    for (let i = 0; i < sentence.length; i += 1) {
      if (sentence[i] === 'F') {
        const newSegment = segment(length, radius, angleX, angleY, angleZ, color, previousLength)
        previousSegment.add(newSegment)
        previousSegment = newSegment
        angleX = 0
        angleY = 0
        angleZ = 0
        previousLength = length
        length *= lengthModifier
        radius *= radialModifier

        if (radius * radialModifier < 0.03) {
          radius = 0.03
        }

        if (length * lengthModifier < 0.3) {
          length = 0.3
        }
      }

      if (sentence[i] === '%') {
        angleX += angle
      }

      if (sentence[i] === '&') {
        angleX -= angle
      }

      if (sentence[i] === '<') {
        angleY += angle
      }

      if (sentence[i] === '>') {
        angleY -= angle
      }

      if (sentence[i] === '+') {
        angleZ += angle
      }

      if (sentence[i] === '-') {
        angleZ -= angle
      }

      if (sentence[i] === '[') {
        states.push({
          segment: previousSegment,
          angleX,
          angleY,
          angleZ,
          previousLength,
          length,
          radius
        })
      }

      if (sentence[i] === ']') {
        const lastState = states[states.length - 1]
        previousSegment = lastState.segment
        angleX = lastState.angleX
        angleY = lastState.angleY
        angleZ = lastState.angleZ
        previousLength = lastState.previousLength
        length = lastState.length
        radius = lastState.radius
        states.pop()
      }
    }
  }

  const createSentence = (iterations: number) => {
    let currentSentence = axiom
    let newSentence = ''

    for (let i = 0; i <= iterations; i += 1) {
      for (let j = 0; j < currentSentence.length; j += 1) {
        if (currentSentence[j] === 'F') {
          newSentence += rule
        } else {
          newSentence += currentSentence[j]
        }
      }

      currentSentence = newSentence
      newSentence = ''
    }

    sentence = currentSentence

    constructScene(sentence)
  }

  createSentence(iterations)

  const renderer = new THREE.WebGLRenderer({ canvas: canvas.value, alpha: true, antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  const controls = new OrbitControls(camera, canvas.value)
  controls.enableDamping = true
  controls.enableZoom = false

  const render = () => {
    renderer.render(scene, camera)

    controls.update()

    requestAnimationFrame(render)
  }

  requestAnimationFrame(render)

  window.addEventListener('resize', () => debounce(resizeCanvas(camera, renderer)))
})
</script>

<template>
  <canvas ref="canvas" />
</template>

<style lang="scss" scoped>
canvas {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
}
</style>
