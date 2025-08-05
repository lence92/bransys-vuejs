<script setup lang="ts">
import vehicles from '@/data/vehicles.json'
import { ref, type Ref } from 'vue'
import VehicleEntryForm from '../components/VehicleEntryForm.vue'

interface Vehicle {
  vehicleName: string
  vehicleType: string
  model: string
  plateNumber: string
}
const filteredVehicles: Ref<Vehicle[]> = ref(vehicles)

const showModal = ref(false)
const editVehicleIndex = ref<number | null>(null)
const editVehicleData = ref<Vehicle | null>(null)
const successMessage = ref('')

const openEditModal = (index: number, vehicle: Vehicle) => {
  editVehicleIndex.value = index
  editVehicleData.value = vehicle
  showModal.value = true
}

const editVehicle = (index: number, vehicle: Vehicle) => {
  // Logic to edit vehicle by index
  console.log(`Edit vehicle with name: ${vehicle.vehicleName}`)
  filteredVehicles.value[index] = {
    vehicleName: vehicle.vehicleName,
    vehicleType: vehicle.vehicleType,
    model: vehicle.model,
    plateNumber: vehicle.plateNumber,
  }
  showModal.value = false
  editVehicleIndex.value = null
  editVehicleData.value = null
  successMessage.value = `Vehicle updated successfully! Name: ${vehicle.vehicleName}`
  setTimeout(() => {
    successMessage.value = ''
  }, 3000)
}

const deleteVehicle = (index: number) => {
  const result = window.confirm(`Are you sure you want to delete vehicle with index: ${index}?`)
  if (result) {
    // Logic to delete vehicle by index
    console.log(`Delete vehicle with index: ${index}`)
    successMessage.value = `Vehicle with index ${index} deleted successfully!`
    setTimeout(() => {
      successMessage.value = ''
    }, 3000)
    // For now, just filter out the vehicle from the list
    filteredVehicles.value = filteredVehicles.value.filter((_, i) => i !== index)
  }
}
</script>

<template>
  <small v-if="successMessage" class="success-message">
    {{ successMessage }}
  </small>

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
      <tr v-for="(vehicle, index) in filteredVehicles" :key="index">
        <td>{{ vehicle.vehicleName }}</td>
        <td>{{ vehicle.vehicleType }}</td>
        <td>{{ vehicle.model }}</td>
        <td>{{ vehicle.plateNumber }}</td>
        <td class="actions">
          <button @click="openEditModal(index, vehicle)">Edit</button>
          <button @click="deleteVehicle(index)">Delete</button>
        </td>
      </tr>
    </tbody>
  </table>

  <!-- The Modal -->
  <div v-if="showModal" id="myModal" class="modal">
    <!-- Modal content -->
    <div class="modal-content">
      <span class="close" @click="showModal = false">&times;</span>
      <VehicleEntryForm
        :vehicleUpdate="editVehicleData"
        @edit-vehicle="editVehicle(editVehicleIndex, $event)"
      />
    </div>
  </div>
</template>

<style>
.success-message {
  color: green;
  font-weight: bold;
  margin-bottom: 1rem;
}

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

/* The Modal (background) */
.modal {
  position: fixed; /* Stay in place */
  z-index: 1; /* Sit on top */
  left: 0;
  top: 0;
  width: 100%; /* Full width */
  height: 100%; /* Full height */
  overflow: auto; /* Enable scroll if needed */
  background-color: rgb(0, 0, 0); /* Fallback color */
  background-color: rgba(0, 0, 0, 0.4); /* Black w/ opacity */
}

/* Modal Content/Box */
.modal-content {
  background-color: #fefefe;
  margin: 15% auto; /* 15% from the top and centered */
  padding: 20px;
  border: 1px solid #888;
  width: 80%; /* Could be more or less, depending on screen size */
}

/* The Close Button */
.close {
  color: #aaa;
  float: right;
  font-size: 28px;
  font-weight: bold;
}

.close:hover,
.close:focus {
  color: black;
  text-decoration: none;
  cursor: pointer;
}
</style>
