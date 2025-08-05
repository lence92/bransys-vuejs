<script setup lang="ts">
import { watch, ref, watchEffect } from 'vue'
import { useForm, useField } from 'vee-validate'
import * as yup from 'yup'

const { vehicleUpdate } = defineProps(['vehicleUpdate'])
const emit = defineEmits(['editVehicle'])
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

const schema = yup.object({
  name: yup
    .string()
    .required('Name is a required field')
    .max(50, 'Name must be at most 50 characters long'),
  type: yup.string().required('Type is a required field'),
  model: yup.string().required('Model is a required field'),
  licensePlate: yup
    .string()
    .required('License plate is a required field')
    .matches(
      /^[A-Z]{2}-[0-9]{4}-[A-Z]{2}$/,
      'License plate must be in format XX-NNNN-XX (e.g., AB-1234-CD)',
    ),
  lastRegistration: yup
    .date()
    .required('Last registration date is a required field')
    .max(getTodayDate(), 'Last registration date cannot be in the future'),
  registrationSubmission: yup
    .date()
    .required('Date of registration submission is a required field')
    .min(
      yup.ref('lastRegistration'),
      'Date of registration submission must be after last registration date',
    ),
})

// Initialize form with schema
const { handleSubmit, errors, meta, isSubmitting } = useForm({ validationSchema: schema })

const { value: name } = useField<string>('name')
const { value: type } = useField<string>('type', schema, {
  initialValue: '',
})
const { value: model } = useField<string>('model')
const { value: licensePlate } = useField<string>('licensePlate')
const { value: lastRegistration } = useField('lastRegistration')
const { value: registrationSubmission } = useField('registrationSubmission', schema, {
  initialValue: getTodayDate(),
})

// Submit handler
const onSubmit = handleSubmit(() => {
  if (vehicleUpdate) {
    emit('editVehicle', {
      vehicleName: name.value,
      vehicleType: type.value,
      model: model.value,
      plateNumber: licensePlate.value,
    })
  } else {
    alert(
      `Vehicle form submitted! Name: ${name.value}, Type: ${type.value}, Model: ${model.value}, License Plate: ${licensePlate.value}, Last Registration: ${lastRegistration.value}, Date of registration submission: ${registrationSubmission.value}`,
    )
  }
})

// Vehicle models autocomplete
const query = ref('')
const filteredOptions = ref([...vehicleModels])
const isAutocompleteTouched = ref(false)

watchEffect(() => {
  if (vehicleUpdate) {
    name.value = vehicleUpdate.vehicleName
    type.value = vehicleUpdate.vehicleType
    query.value = vehicleUpdate.model
    licensePlate.value = vehicleUpdate.plateNumber
  }
})

watch(
  query,
  (newValue) => {
    model.value = newValue // update model in formData when query changes
  },
  { immediate: vehicleUpdate },
)

const filterOptions = () => {
  const search = query.value.toLowerCase()
  filteredOptions.value = vehicleModels.filter((option) => option.toLowerCase().includes(search))
}

const selectOption = (option: string) => {
  query.value = option // set input to selected option
  filteredOptions.value = [] // hide suggestions
}

// License plate formatting and validation
const formatLicensePlate = (event: Event) => {
  let value = (event.target as HTMLInputElement).value.toUpperCase().replace(/[^A-Z0-9]/g, '') // Remove non-alphanumeric and convert to uppercase

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

  licensePlate.value = value
}
</script>

<template>
  <form @submit.prevent="onSubmit">
    <h2>Vehicle Entry form</h2>
    <div>
      <label for="name">Name:</label>
      <input type="text" id="name" v-model="name" maxlength="50" />
      <small class="error">{{ errors.name }}</small>
    </div>
    <div>
      <label for="type">Type:</label>
      <select id="type" v-model="type">
        <option value="">Select type</option>
        <option v-for="type in vehicleTypes" :key="type" :value="type">
          {{ type }}
        </option>
      </select>
      <small class="error">{{ errors.type }}</small>
    </div>
    <div class="autocomplete">
      <label for="model">Model:</label>
      <input
        type="text"
        id="model"
        v-model="query"
        @input="filterOptions"
        @keyup="isAutocompleteTouched = true"
        placeholder="Type to search and select a model..."
      />

      <ul
        v-if="isAutocompleteTouched && query !== '' && filteredOptions.length > 0"
        class="suggestions"
      >
        <li v-for="(option, index) in filteredOptions" :key="index" @click="selectOption(option)">
          {{ option }}
        </li>
      </ul>
      <small class="error">{{ errors.model }}</small>
    </div>
    <div>
      <label for="licensePlate">License Plate number:</label>
      <input
        type="text"
        id="licensePlate"
        :value="licensePlate"
        @input="formatLicensePlate"
        placeholder="XX-NNNN-XX"
        maxlength="10"
      />
      <small class="error">
        {{ errors.licensePlate }}
      </small>
    </div>
    <div>
      <label for="lastRegistration">Last Registration date:</label>
      <input type="date" id="lastRegistration" v-model="lastRegistration" :max="getTodayDate()" />
      <small class="error">
        {{ errors.lastRegistration }}
      </small>
    </div>
    <div>
      <label for="registrationSubmission">Date of registration submission:</label>
      <input
        type="date"
        id="registrationSubmission"
        v-model="registrationSubmission"
        :min="lastRegistration ? lastRegistration : getTodayDate()"
      />
      <small class="error">
        {{ errors.registrationSubmission }}
      </small>
    </div>
    <button type="submit" :disabled="!meta.valid || isSubmitting">
      {{ vehicleUpdate ? 'Update' : 'Create' }} Vehicle
    </button>
  </form>
</template>

<style>
.error {
  color: red;
}

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
