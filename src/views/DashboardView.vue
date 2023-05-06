<template>
  <div>
    <LoginForm v-if="!isLoggedIn" @login="isLoggedIn = true"/>
    <div v-else >
      <div class="btn-group mt-4 d-flex justify-content-center">
        <button class="btn btn-primary toggle-button" :class="{ 'active': role === 'patient' }" @click="role = 'patient'">Patient</button>
        <button class="btn btn-primary toggle-button" :class="{ 'active': role === 'doctor' }" @click="role = 'doctor'">Doctor</button>
      </div>
      <div>
        <DoctorList :doctors="doctors" :specialties="specialties" @add-doctor="addDoctor" @delete-doctor="deleteDoctor" v-show="role === 'doctor'"/>
        <PatientList :patients="patients" @add-patient="addPatient" @delete-patient="deletePatient" v-show="role === 'patient'"/>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import DoctorList from '@/components/DoctorList.vue'
import PatientList from '@/components/PatientList.vue'
import LoginForm from '@/components/LoginForm.vue'

export default {
  name: 'DashboardPage',
  components: {
    DoctorList,
    PatientList,
    LoginForm
  },
  data () {
    return {
      isLoggedIn: false,
      role: 'doctor',
      doctors: [],
      patients: [],
      specialties: []
    }
  },
  mounted () {
    axios.get('http://localhost:3000/doctors').then(response => {
      this.doctors = response.data
    })
    axios.get('http://localhost:3000/patients').then(response => {
      this.patients = response.data
    })
    axios.get('http://localhost:3000/specialties').then(response => {
      this.specialties = response.data
    })
  },
  methods: {
    addDoctor (doctor) {
      this.doctors.push(doctor)
      return null
    },
    deleteDoctor (doctorId) {
      this.doctors = this.doctors.filter(d => d.id !== doctorId)
      return null
    },
    addPatient (patient) {
      this.patients.push(patient)
      return null
    },
    deletePatient (patientId) {
      this.patients = this.patients.filter(p => p.id !== patientId)
      return null
    },
    toggleRole () {
      this.role = this.role === 'doctor' ? 'patient' : 'doctor'
    }
  }
}
</script>

<style>
.toggle-button {
  border-radius: 15px;
  cursor: pointer;
  position: relative;
  font-size: 16px;
  height: 30px;
  padding: 0 20px;
}

.toggle-button.active {
  background-color: rgb(16, 180, 16);
}

.toggle-button:not(.active) {
  background-color: #032511;
  color: white;
}

.toggle-button:not(:first-child) {
  margin-left: 10px;
}

.toggle-button:focus {
  outline: none;
  box-shadow: none;
}
</style>
