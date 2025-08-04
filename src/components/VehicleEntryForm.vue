<script setup lang="ts">
import { watch, ref } from 'vue'

const vehicleTypes = ['Car', 'Van', 'Truck', 'Container', 'Trailer', 'Dumper']
const vehicleModels = [
  'Audi',
  'BMW',
  'Citroen',
  'Honda',
  'Hyundai',
  'Mercedes',
  'Nissan',
  'Opel',
  'Suzuki',
  'Toyota',
  'Volkswagen',
]
// Get today's date in YYYY-MM-DD format
const getTodayDate = () => {
  const today = new Date()
  return today.toISOString().split('T')[0]
}

const formData = ref({
  name: '',
  type: '',
  model: '',
  licensePlate: '',
  lastRegistration: '',
  registrationSubmission: getTodayDate(),
})

const isFormValid = ref(false)

watch(
  formData,
  (newValue) => {
    console.log('newValue:', newValue)
    isFormValid.value =
      newValue.name !== '' &&
      newValue.type !== '' &&
      newValue.licensePlate !== '' &&
      newValue.lastRegistration !== '' &&
      newValue.registrationSubmission !== '' &&
      validateLicensePlate()
  },
  { deep: true },
)

const submitForm = () => {
  console.log('Form submitted:', formData.value)
}

const query = ref('')

watch(query, (newValue) => {
  console.log('Query changed:', newValue)
  formData.value.model = newValue // update model in formData when query changes
})

// Vehicle models autocomplete
const filteredOptions = ref([...vehicleModels])

const filterOptions = () => {
  const search = query.value.toLowerCase()
  filteredOptions.value = vehicleModels.filter((option) => option.toLowerCase().includes(search))
}

const selectOption = (option) => {
  query.value = option // set input to selected option
  filteredOptions.value = [] // hide suggestions
}

// License plate formatting and validation
const formatLicensePlate = (event) => {
  let value = event.target.value.toUpperCase().replace(/[^A-Z0-9]/g, '') // Remove non-alphanumeric and convert to uppercase

  // Apply formatting: XX-NNNN-XX
  if (value.length > 0) {
    if (value.length <= 2) {
      // First section: up to 2 letters
      value = value.replace(/[^A-Z]/g, '')
    } else if (value.length <= 6) {
      // Second section: 4 numbers
      const letters = value.slice(0, 2).replace(/[^A-Z]/g, '')
      const numbers = value.slice(2, 6).replace(/[^0-9]/g, '')
      value = letters + (numbers ? '-' + numbers : '')
    } else {
      // Third section: 2 letters
      const letters1 = value.slice(0, 2).replace(/[^A-Z]/g, '')
      const numbers = value.slice(2, 6).replace(/[^0-9]/g, '')
      const letters2 = value.slice(6, 8).replace(/[^A-Z]/g, '')
      value = letters1 + '-' + numbers + (letters2 ? '-' + letters2 : '')
    }
  }

  formData.value.licensePlate = value
}

const validateLicensePlate = () => {
  const pattern = /^[A-Z]{2}-[0-9]{4}-[A-Z]{2}$/
  return pattern.test(formData.value.licensePlate)
}
</script>

<template>
  <form @submit.prevent="submitForm">
    <h2>Vehicle Entry Form</h2>
    <div>
      <label for="name">Name:</label>
      <input type="text" id="name" v-model="formData.name" required maxlength="50" />
    </div>
    <div>
      <label for="type">Type:</label>
      <select id="type" v-model="formData.type" required>
        <option value="" disabled>Select type</option>
        <option v-for="type in vehicleTypes" :key="type" :value="type.toLowerCase()">
          {{ type }}
        </option>
      </select>
    </div>
    <div class="autocomplete">
      <label for="model">Model:</label>
      <input
        type="text"
        id="model"
        v-model="query"
        @input="filterOptions"
        placeholder="Type to search and select a model..."
      />

      <ul v-if="query && filteredOptions.length > 0" class="suggestions">
        <li v-for="(option, index) in filteredOptions" :key="index" @click="selectOption(option)">
          {{ option }}
        </li>
      </ul>
    </div>
    <div>
      <label for="licensePlate">License Plate number:</label>
      <input
        type="text"
        id="licensePlate"
        :value="formData.licensePlate"
        @input="formatLicensePlate"
        placeholder="XX-NNNN-XX"
        maxlength="10"
        required
      />
      <small v-if="formData.licensePlate && !validateLicensePlate()" style="color: red">
        Invalid format. Use XX-NNNN-XX (e.g., AB-1234-CD)
      </small>
    </div>
    <div>
      <label for="lastRegistration">Last Registration date:</label>
      <input
        type="date"
        id="lastRegistration"
        v-model="formData.lastRegistration"
        :max="getTodayDate()"
        required
      />
    </div>
    <div>
      <label for="registrationSubmission">Date of registration submission:</label>
      <input
        type="date"
        id="registrationSubmission"
        v-model="formData.registrationSubmission"
        :min="formData.lastRegistration"
        required
      />
    </div>
    <button type="submit" :disabled="!isFormValid">Submit</button>
  </form>
</template>

<style>
form div {
  margin-bottom: 15px;
}

input,
select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
}

button {
  padding: 8px 16px;
  background-color: var(--vt-c-green);
  color: white;
  border: none;
  cursor: pointer;
}
button:hover {
  background-color: var(--vt-c-green-dark);
}
button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.autocomplete {
  position: relative;
  width: 250px;
}

.suggestions {
  list-style: none;
  margin: 0;
  padding: 0;
  border: 1px solid #ccc;
  border-top: none;
  max-height: 150px;
  overflow-y: auto;
  position: absolute;
  width: 100%;
  background: white;
  z-index: 10;
}
.suggestions li {
  padding: 8px;
  cursor: pointer;
}
.suggestions li:hover {
  background: #f0f0f0;
}
</style>
