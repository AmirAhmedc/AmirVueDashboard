<template>
  <div class="container">
    <h1>Doctors</h1>
    <div class="table-responsive">
      <table class="table">
        <thead>
          <tr>
            <th>Name</th>
            <th>Phone</th>
            <th>Specialties</th>
            <th></th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="doctor in doctors" :key="doctor.id">
            <td>{{ doctor.name }}</td>
            <td>{{ doctor.phone }}</td>
            <td>{{ doctor.specialties.join(', ') }}</td>
            <td>
              <button @click="editDoctor(doctor)" class="btn btn-primary">Edit</button>
            </td>
            <td>
              <button @click="deleteDoctor(doctor.id)" class="btn btn-danger">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <form v-if="editingDoctor" @submit.prevent="updateDoctor" @reset="resetForm">
      <div class="form-group has-feedback" :class="{'has-error': !isNameValid}">
        <label for="name">Name:</label>
        <input type="text" class="form-control" id="name" v-model.trim="editingDoctor.name">
        <span v-if="!isNameValid" class="help-block"><i class="glyphicon glyphicon-remove form-control-feedback"></i>Name is required and must be a string.</span>
      </div>
      <div class="form-group has-feedback" :class="{'has-error': !isPhoneValid}">
        <label for="phone">Phone:</label>
        <input type="text" class="form-control" id="phone" v-model.trim="editingDoctor.phone">
        <span v-if="!isPhoneValid" class="help-block"><i class="glyphicon glyphicon-remove form-control-feedback"></i>Phone is required and must be numeric.</span>
      </div>
      <div class="form-group has-feedback" :class="{'has-error': !isSpecialtiesValid}">
        <label for="specialties">Specialties:</label>
        <select multiple class="form-control" id="specialties" v-model="editingDoctor.specialties">
          <option v-for="specialty in specialties" :value="specialty.name" :key="specialty.id">{{ specialty.name }}
          </option>
        </select>
        <span v-if="!isSpecialtiesValid" class="help-block"><i class="glyphicon glyphicon-remove"></i>At least one specialty is required.</span>
      </div>
      <button type="submit" class="btn btn-primary" :disabled="!isFormValid">Update Doctor</button>
      <button type="button" class="btn btn-default" @click="cancelEdit">Cancel</button>
    </form>
    <form v-else @submit.prevent="addDoctor" @reset="resetForm">
      <div class="form-group has-feedback" :class="{'has-error': !isNameValid}">
        <label for="name">Name:</label>
        <input type="text" class="form-control" id="name" v-model.trim="newDoctor.name" required>
        <span v-if="!isNameValid" class="help-block"><i class="glyphicon glyphicon-remove form-control-feedback"></i>Name is required and must be a string.</span>
      </div>
      <div class="form-group has-feedback" :class="{'has-error': !isPhoneValid}">
        <label for="phone">Phone:</label>
        <input type="text" class="form-control" id="phone" v-model.trim="newDoctor.phone">
        <span v-if="!isPhoneValid" class="help-block"><i class="glyphicon glyphicon-remove form-control-feedback"></i>Phone is required and must be numeric.</span>
      </div>
      <div class="form-group has-feedback" :class="{'has-error': !isSpecialtiesValid}">
        <label for="specialties">Specialties:</label>
        <select multiple class="form-control" id="specialties" v-model="newDoctor.specialties">
          <option v-for="specialty in specialties" :value="specialty.name" :key="specialty.id">{{ specialty.name }}
          </option>
        </select>
        <span v-if="!isSpecialtiesValid" class="help-block"><i class="glyphicon glyphicon-remove"></i>At least one specialty is required.</span>
      </div>
      <button type="submit" class="btn btn-primary" :disabled="!isFormValid">Add Doctor</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  props: {
    doctors: {
      type: Array,
      required: true
    },
    specialties: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      newDoctor: {
        name: '',
        phone: '',
        specialties: []
      },
      editingDoctor: null,
      formSubmitted: false,
      selectedDoctorIndex: null

    }
  },
  methods: {
    addDoctor () {
      axios.post('http://localhost:3000/doctors', this.newDoctor)
        .then(response => {
          this.$emit('add-doctor', response.data)
          this.newDoctor = {
            name: '',
            phone: '',
            specialties: []
          }
          this.formSubmitted = true
        })
        .catch(error => console.log(error))
    },
    deleteDoctor (doctorId) {
      axios.delete(`http://localhost:3000/doctors/${doctorId}`)
        .then(response => {
          this.$emit('delete-doctor', doctorId)
        })
        .catch(error => console.log(error))
    },
    editDoctor (doctor) {
      this.editingDoctor = { ...doctor }
      this.selectedDoctorIndex = this.doctors.findIndex(
        (d) => d.id === doctor.id
      )
    },
    updateDoctor () {
      axios.put(`http://localhost:3000/doctors/${this.editingDoctor.id}`, this.editingDoctor)
        .then(response => {
          this.$emit('update-doctor', response.data)
          // eslint-disable-next-line vue/no-mutating-props
          this.doctors.splice(this.selectedDoctorIndex, 1, response.data)
          this.editingDoctor = null
          this.formSubmitted = true
        })
        .catch(error => console.log(error))
    },
    cancelEdit () {
      this.editingDoctor = null
      this.formSubmitted = false
    },
    resetForm () {
      this.newDoctor = {
        name: '',
        phone: '',
        specialties: []
      }
      this.editingDoctor = null
      this.formSubmitted = false
    }
  },
  computed: {
    nameValue () {
      return this.editingDoctor ? this.editingDoctor.name : this.newDoctor.name
    },
    isNameValid () {
      const name = this.nameValue
      return name !== '' && typeof name === 'string' && !/\d/.test(name)
    },
    phoneValue () {
      return this.editingDoctor ? this.editingDoctor.phone : this.newDoctor.phone
    },
    isPhoneValid () {
      const phone = this.phoneValue
      return phone !== '' && typeof phone === 'string' && /^[0-9()+-]*$/.test(phone)
    },
    isSpecialtiesValid () {
      return (this.formSubmitted || this.newDoctor.specialties.length > 0 || (this.editingDoctor && this.editingDoctor.specialties.length > 0))
    },
    isFormValid () {
      if (this.editingDoctor) {
        return this.isNameValid && this.isPhoneValid && this.isSpecialtiesValid && this.editingDoctor.name !== '' && this.editingDoctor.phone !== ''
      } else {
        return this.isNameValid && this.isPhoneValid && this.isSpecialtiesValid
      }
    }
  },
  watch: {
    newDoctor: {
      handler () {
        this.formSubmitted = false
      },
      deep: true
    },
    editingDoctor: {
      handler () {
        this.formSubmitted = false
      },
      deep: true
    }
  }
}
</script>

<style>
  .has-error .help-block {
    color: #a94442; /* Red color */
  }
  .form-control-feedback {
    right: 15px;
    top: 8px;
    pointer-events: none;
    position: absolute;
  }
</style>
