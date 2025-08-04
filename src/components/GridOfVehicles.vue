<script setup lang="ts">
import vehicles from '@/data/vehicles.json'
</script>
<template>
  <table>
    <thead>
      <tr>
        <th>Vehicle name</th>
        <th>Vehicle type</th>
        <th>Model</th>
        <th>License plate number</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="vehicle in vehicles" :key="vehicle.id">
        <td>{{ vehicle.vehicleName }}</td>
        <td>{{ vehicle.vehicleType }}</td>
        <td>{{ vehicle.model }}</td>
        <td>{{ vehicle.plateNumber }}</td>
        <td class="actions">
          <button @click="editVehicle(vehicle.id)">Edit</button>
          <button @click="deleteVehicle(vehicle.id)">Delete</button>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<style>
table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
  text-align: center;
}

.actions {
  display: flex;
  gap: 0.5rem;
}
</style>
