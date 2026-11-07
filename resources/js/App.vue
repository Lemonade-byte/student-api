<template>
  <div class="max-w-4xl mx-auto p-6 font-sans">
    <header class="flex justify-between items-center border-b pb-4 mb-6">
      <h1 class="text-2xl font-bold text-gray-800">Student Management System</h1>
      
      <!-- Login & User Status -->
      <div>
        <div v-if="!token" class="flex gap-2">
          <button 
            @click="login('admin@example.com')" 
            class="px-3 py-1 bg-purple-600 text-white rounded text-sm hover:bg-purple-700"
          >
            Login as Admin
          </button>
          <button 
            @click="login('user@example.com')" 
            class="px-3 py-1 bg-gray-600 text-white rounded text-sm hover:bg-gray-700"
          >
            Login as User
          </button>
        </div>
        <div v-else class="flex items-center gap-3">
          <span class="text-sm font-medium text-green-700">Logged in ({{ userRole }})</span>
          <button 
            @click="logout" 
            class="px-3 py-1 bg-red-500 text-white rounded text-sm hover:bg-red-600"
          >
            Logout
          </button>
        </div>
      </div>
    </header>

    <!-- Feedback / Validation Alert -->
    <div 
      v-if="message" 
      :class="isError ? 'bg-red-100 text-red-700 border-red-400' : 'bg-green-100 text-green-700 border-green-400'"
      class="p-3 border rounded mb-4 flex justify-between items-center"
    >
      <span>{{ message }}</span>
      <button @click="message = ''" class="font-bold ml-2">&times;</button>
    </div>

    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <!-- Student Form -->
      <div class="md:col-span-1">
        <StudentForm 
          :editing-student="selectedStudent" 
          @save-student="handleSave"
          @cancel-edit="selectedStudent = null"
        />
      </div>

      <!-- Student List -->
      <div class="md:col-span-2">
        <StudentList 
          :students="students" 
          @edit-student="setEditing" 
          @delete-student="handleDelete"
        />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import StudentList from './components/StudentList.vue';
import StudentForm from './components/StudentForm.vue';

export default {
  components: { StudentList, StudentForm },
  data() {
    return {
      students: [],
      selectedStudent: null,
      token: localStorage.getItem('token') || '',
      userRole: localStorage.getItem('role') || '',
      message: '',
      isError: false
    }
  },
  mounted() {
    if (this.token) {
      axios.defaults.headers.common['Authorization'] = `Bearer ${this.token}`;
      this.fetchStudents();
    }
  },
  methods: {
    showMessage(msg, error = false) {
      this.message = msg;
      this.isError = error;
      setTimeout(() => { this.message = ''; }, 4000);
    },

    async login(email) {
      try {
        const res = await axios.post('/api/login', {
          email,
          password: 'password123'
        });
        this.token = res.data.access_token;
        this.userRole = res.data.role;
        localStorage.setItem('token', this.token);
        localStorage.setItem('role', this.userRole);
        axios.defaults.headers.common['Authorization'] = `Bearer ${this.token}`;
        this.showMessage(`Logged in successfully as ${this.userRole}!`);
        this.fetchStudents();
      } catch (err) {
        this.showMessage('Login failed!', true);
      }
    },

    async logout() {
      try {
        await axios.post('/api/logout');
      } catch (e) {
        // Continue clearing session even if token is already expired
      }
      this.token = '';
      this.userRole = '';
      this.students = [];
      localStorage.removeItem('token');
      localStorage.removeItem('role');
      delete axios.defaults.headers.common['Authorization'];
      this.showMessage('Logged out successfully.');
    },

    async fetchStudents() {
      if (!this.token) return;
      try {
        const res = await axios.get('/api/students');
        this.students = res.data;
      } catch (err) {
        this.showMessage(err.response?.data?.message || 'Failed to fetch students.', true);
      }
    },

    async handleSave(formData) {
      if (!this.token) {
        this.showMessage('Please login first!', true);
        return;
      }
      try {
        if (this.selectedStudent) {
          await axios.put(`/api/students/${this.selectedStudent.id}`, formData);
          this.showMessage('Student updated successfully!');
          this.selectedStudent = null;
        } else {
          await axios.post('/api/students', formData);
          this.showMessage('Student created successfully!');
        }
        this.fetchStudents();
      } catch (err) {
        this.showMessage(err.response?.data?.message || 'Operation failed. Admin role required.', true);
      }
    },

    async handleDelete(id) {
      if (!confirm('Are you sure you want to delete this student?')) return;
      try {
        await axios.delete(`/api/students/${id}`);
        this.showMessage('Student deleted successfully!');
        this.fetchStudents();
      } catch (err) {
        this.showMessage(err.response?.data?.message || 'Delete failed. Admin role required.', true);
      }
    },

    setEditing(student) {
      this.selectedStudent = student;
    }
  }
}
</script>