<template>
  <div class="container">
    <h1>Patients</h1>
    <div class="table-responsive">
      <table class="table">
        <thead>
          <tr>
            <th>Name</th>
            <th>Phone</th>
            <th>File Name</th>
            <th></th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="patient in patients" :key="patient.id">
            <td>{{ patient.name }}</td>
            <td>{{ patient.phone }}</td>
            <td><a :href="patient.fileUrl">{{ patient.fileName }}</a></td>
            <td><button @click="deletePatient(patient.id)" class="btn btn-danger">Delete</button></td>
            <td><button @click="editPatient(patient)" class="btn btn-primary">Edit</button></td>
          </tr>
        </tbody>
      </table>
    </div>
    <div v-if="!editing">
      <form @submit.prevent="addPatient" @reset="resetForm">
        <div class="form-group has-feedback" :class="{'has-error': !isNameValid}">
          <label for="name">Name:</label>
          <input type="text" class="form-control" id="name" v-model.trim="newPatient.name">
          <span v-if="!isNameValid" class="help-block"><i class="glyphicon glyphicon-remove form-control-feedback"></i>Name is required and must be a string.</span>
        </div>
        <div class="form-group has-feedback" :class="{'has-error': !isPhoneValid}">
          <label for="phone">Phone:</label>
          <input type="text" class="form-control" id="phone" v-model.trim="newPatient.phone">
          <span v-if="!isPhoneValid" class="help-block"><i class="glyphicon glyphicon-remove form-control-feedback"></i>Phone is required and must be numeric.</span>
        </div>
        <div class="form-group has-feedback">
          <label for="file">File:</label>
          <input type="file" class="form-control-file" id="file" accept="image/*,.pdf" @change="handleFileUpload">
          <div v-if="newPatient.filePreviewUrl">
            <img :src="newPatient.filePreviewUrl" v-if="newPatient.file.type.startsWith('image/')">
            <a :href="newPatient.filePreviewUrl" v-else>{{ newPatient.fileName }}</a>
          </div>
        </div>
        <button type="submit" class="btn btn-primary" :disabled="!isFormValid">Add Patient</button>
      </form>
    </div>
    <div v-if="editing">
      <form @submit.prevent="updatePatient" @reset="resetForm">
        <div class="form-group has-feedback" :class="{'has-error': !isNameValid}">
          <label for="name">Name:</label>
          <input type="text" class="form-control" id="name" v-model.trim="selectedPatient.name">
          <span v-if="!isNameValid" class="help-block"><i class="glyphicon glyphicon-remove form-control-feedback"></i>Name is required and must be a string.</span>
        </div>
        <div class="form-group has-feedback" :class="{'has-error': !isPhoneValid}">
          <label for="phone">Phone:</label>
          <input type="text" class="form-control" id="phone" v-model.trim="selectedPatient.phone">
          <span v-if="!isPhoneValid" class="help-block"><i class="glyphicon glyphicon-remove form-control-feedback"></i>Phone is required and must be numeric.</span>
        </div>
        <div class="form-group has-feedback">
          <label for="file">File:</label>
          <input type="file" class="form-control-file" id="file" accept="image/*,.pdf" @change="handleFileUpload">
          <div v-if="selectedPatient.filePreviewUrl">
            <img :src="selectedPatient.filePreviewUrl" v-if="selectedPatient.file.type.startsWith('image/')">
            <a :href="selectedPatient.filePreviewUrl" v-else>{{ selectedPatient.fileName }}</a>
          </div>
        </div>
        <button type="submit" class="btn btn-primary">Update Patient</button>
        <button @click="cancelEdit" class="btn btn-secondary">Cancel</button>
      </form>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data () {
    return {
      patients: [],
      newPatient: {
        name: '',
        phone: '',
        file: null,
        fileUrl: '',
        fileName: '',
        fileData: '',
        filePreviewUrl: ''
      },
      selectedPatient: null,
      formSubmitted: false,
      editing: false
    }
  },
  mounted () {
    this.fetchPatients()
  },
  methods: {
    async fetchPatients () {
      try {
        const { data } = await axios.get('http://localhost:3000/patients')
        this.patients = data
      } catch (error) {
        console.log(error)
      }
    },

    async addPatient () {
      const { file } = this.newPatient

      try {
        const base64EncodedFile = await this.readFile(file)
        const formData = {
          name: this.newPatient.name,
          phone: this.newPatient.phone,
          fileUrl: `/files/${file.name}`,
          fileName: file.name,
          fileData: base64EncodedFile
        }

        const { data } = await axios.post('http://localhost:3000/patients', formData)
        this.patients.push(data)
        this.formSubmitted = true
        this.resetNewPatient()
      } catch (error) {
        console.log(error)
      }
    },

    editPatient (patient) {
      this.selectedPatient = Object.assign({}, patient)
      this.editing = true
      this.newPatient.filePreviewUrl = patient.fileUrl
    },

    async updatePatient () {
      const { selectedPatient } = this
      const { file } = this.newPatient

      try {
        let formData = {
          name: selectedPatient.name,
          phone: selectedPatient.phone,
          fileName: selectedPatient.fileName,
          fileUrl: selectedPatient.fileUrl,
          fileData: null
        }

        if (file) {
          const base64EncodedFile = await this.readFile(file)
          formData = {
            ...formData,
            fileName: file.name,
            fileUrl: `/files/${file.name}`,
            fileData: base64EncodedFile
          }
        }

        const { data } = await axios.put(
          `http://localhost:3000/patients/${selectedPatient.id}`,
          formData
        )
        const index = this.patients.findIndex((p) => p.id === data.id)
        if (index !== -1) {
          this.patients.splice(index, 1, data)
        }

        this.selectedPatient = null
        this.editing = false
        this.resetNewPatient()
        this.formSubmitted = true
      } catch (error) {
        console.log(error)
      }
    },

    deletePatient (patientId) {
      axios.delete(`http://localhost:3000/patients/${patientId}`)
        .then(response => {
          const index = this.patients.findIndex(p => p.id === patientId)
          if (index !== -1) {
            this.patients.splice(index, 1)
          }
        })
        .catch(error => console.log(error))
    },

    handleFileUpload (event) {
      const file = event.target.files[0]
      this.newPatient.file = file

      const reader = new FileReader()
      reader.onload = () => {
        this.newPatient.filePreviewUrl = reader.result
      }
      if (file.type.startsWith('image/')) {
        reader.readAsDataURL(file)
      } else if (file.type === 'application/pdf') {
        this.newPatient.filePreviewUrl = URL.createObjectURL(file)
      }
    },

    resetNewPatient () {
      this.newPatient = {
        name: '',
        phone: '',
        file: null,
        fileUrl: '',
        fileName: '',
        fileData: '',
        filePreviewUrl: ''
      }
      this.formSubmitted = false
    },

    cancelEdit () {
      this.selectedPatient = null
      this.editing = false
      this.resetNewPatient()
      this.formSubmitted = false
    },

    async readFile (file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = () => {
          resolve(reader.result.split(',')[1])
        }
        reader.onerror = (error) => {
          reject(error)
        }
        reader.readAsDataURL(file)
      })
    }
  },
  computed: {
    nameValue () {
      return this.selectedPatient ? this.selectedPatient.name : this.newPatient.name
    },
    isNameValid () {
      const name = this.nameValue
      return name !== '' && typeof name === 'string' && !/\d/.test(name)
    },
    phoneValue () {
      return this.selectedPatient ? this.selectedPatient.phone : this.newPatient.phone
    },
    isPhoneValid () {
      const phone = this.phoneValue
      return phone !== '' && typeof phone === 'string' && /^[0-9()+-]*$/.test(phone)
    },
    isFormValid () {
      if (this.selectedPatient) {
        return this.isNameValid && this.isPhoneValid && this.selectedPatient.name !== '' && this.selectedPatient.phone !== ''
      } else {
        return this.isNameValid && this.isPhoneValid
      }
    }
  },
  watch: {
    newPatient: {
      handler () {
        this.formSubmitted = false
      },
      deep: true
    },
    selectedPatient: {
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
