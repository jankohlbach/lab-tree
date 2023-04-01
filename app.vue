<script setup lang="ts">
import Lenis from '@studio-freight/lenis'

onMounted(() => {
  const lenis = new Lenis({ lerp: 0.075 })

  const raf = (time: number) => {
    lenis.raf(time)
    requestAnimationFrame(raf)
  }

  requestAnimationFrame(raf)
})
</script>

<template>
  <main>
    <Tree />
  </main>
</template>
