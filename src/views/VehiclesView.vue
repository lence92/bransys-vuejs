<script setup lang="ts">
import GridOfVehicles from '@/components/GridOfVehicles.vue'
</script>

<template>
  <div class="vehicles">
    <GridOfVehicles />
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .vehicles {
    /* min-height: 100vh; */
    display: flex;
    align-items: center;
  }
}
</style>
