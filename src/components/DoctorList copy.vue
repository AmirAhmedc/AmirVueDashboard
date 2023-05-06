<template>
  <div>
    <LoginForm v-if="!isLoggedIn" @login="isLoggedIn = true"/>
    <div v-else >
      <div class="toggle-container mt-4 text-center">
        <button class="toggle-button outline-0" :class="{ 'toggle-left': role === 'patient', 'toggle-right': role === 'doctor' }" @click="toggleRole">
          <div class="toggle-label" :style="{ 'margin': role === 'doctor' ? '0 10px 0 -10px' : '0 -10px 0 10px' }">{{ role === 'doctor' ? 'Switch to Patient' : 'Switch to Doctor' }}</div>
          <div class="toggle-circle"></div>
        </button>
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
  font-size: 16px;
  color: white;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  background: #032511;
  border: none;
  padding: 0;
  width: 22%;
  height: 30px;
  border-radius: 15px;
  cursor: pointer;
  position: relative;
}

.toggle-label {
  color: #FFFFFF;
  font-size: 15px;
}

.toggle-circle {
  position: absolute;
  top: 2px;
  left: 15px;
  width: 26px;
  height: 26px;
  border-radius: 50%;
  background-color: #FFFFFF;
}

.toggle-left .toggle-circle {
  left: 2px;
  transform: translateX(0);
  background-color: #FFC700;
}

.toggle-right .toggle-circle {
  right: 2px;
  transform: translateX(112px);
  background-color: rgb(16, 180, 16);
}
</style>
