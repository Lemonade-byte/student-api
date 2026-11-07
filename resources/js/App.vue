<template>
  <div class="max-w-4xl mx-auto p-6 font-sans">
    <header class="flex justify-between items-center border-b pb-4 mb-6">
      <div>
        <h1 class="text-2xl font-bold text-gray-800">Student Management System</h1>
        <p class="text-xs text-gray-500 font-mono mt-1">Vue 3 Composition API + Laravel Sanctum</p>
      </div>
      
      <!-- Login & User Status -->
      <div>
        <div v-if="!token" class="flex gap-2">
          <button 
            @click="login('admin@example.com')" 
            :disabled="loading"
            class="px-3 py-1 bg-purple-600 text-white rounded text-sm hover:bg-purple-700 disabled:opacity-50"
          >
            Login as Admin
          </button>
          <button 
            @click="login('user@example.com')" 
            :disabled="loading"
            class="px-3 py-1 bg-gray-600 text-white rounded text-sm hover:bg-gray-700 disabled:opacity-50"
          >
            Login as User
          </button>
        </div>
        <div v-else class="flex items-center gap-3">
          <span class="text-sm font-semibold px-2.5 py-1 rounded" :class="userRole === 'admin' ? 'bg-purple-100 text-purple-800' : 'bg-blue-100 text-blue-800'">
            Role: {{ userRole.toUpperCase() }}
          </span>
          <button 
            @click="logout" 
            class="px-3 py-1 bg-red-500 text-white rounded text-sm hover:bg-red-600"
          >
            Logout
          </button>
        </div>
      </div>
    </header>

    <!-- Global API Loading Bar -->
    <div v-if="loading" class="p-2.5 bg-blue-50 text-blue-700 border border-blue-200 rounded mb-4 text-sm font-medium animate-pulse flex items-center gap-2">
      <svg class="animate-spin h-4 w-4 text-blue-600" fill="none" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z"></path>
      </svg>
      Connecting to Laravel API...
    </div>

    <!-- Alert / Feedback Message -->
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
          :user-role="userRole"
          :submitting="submitting"
          @save-student="handleSave"
          @cancel-edit="selectedStudent = null"
        />
      </div>

      <!-- Student List -->
      <div class="md:col-span-2">
        <StudentList 
          :students="students" 
          :user-role="userRole"
          :loading="loading"
          @edit-student="setEditing" 
          @delete-student="handleDelete"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import StudentList from './components/StudentList.vue';
import StudentForm from './components/StudentForm.vue';

// Composition API Reactive State using ref()
const students = ref([]);
const selectedStudent = ref(null);
const token = ref(localStorage.getItem('token') || '');
const userRole = ref(localStorage.getItem('role') || '');
const message = ref('');
const isError = ref(false);
const loading = ref(false);
const submitting = ref(false);

const showMessage = (msg, error = false) => {
  message.value = msg;
  isError.value = error;
  setTimeout(() => { message.value = ''; }, 4000);
};

onMounted(() => {
  if (token.value) {
    axios.defaults.headers.common['Authorization'] = `Bearer ${token.value}`;
    fetchStudents();
  }
});

const login = async (email) => {
  loading.value = true;
  try {
    const res = await axios.post('/api/login', {
      email,
      password: 'password123'
    });
    token.value = res.data.access_token;
    userRole.value = res.data.role;
    localStorage.setItem('token', token.value);
    localStorage.setItem('role', userRole.value);
    axios.defaults.headers.common['Authorization'] = `Bearer ${token.value}`;
    showMessage(`Logged in successfully as ${userRole.value.toUpperCase()}!`);
    await fetchStudents();
  } catch (err) {
    showMessage(err.response?.data?.message || 'Login failed!', true);
  } finally {
    loading.value = false;
  }
};

const logout = async () => {
  loading.value = true;
  try {
    await axios.post('/api/logout');
  } catch (e) {
    // Session cleanup fallback
  } finally {
    token.value = '';
    userRole.value = '';
    students.value = [];
    selectedStudent.value = null;
    localStorage.removeItem('token');
    localStorage.removeItem('role');
    delete axios.defaults.headers.common['Authorization'];
    loading.value = false;
    showMessage('Logged out successfully and token revoked.');
  }
};

const fetchStudents = async () => {
  if (!token.value) return;
  loading.value = true;
  try {
    const res = await axios.get('/api/students');
    students.value = res.data;
  } catch (err) {
    showMessage(err.response?.data?.message || 'Failed to fetch students.', true);
  } finally {
    loading.value = false;
  }
};

const handleSave = async (formData) => {
  if (!token.value) {
    showMessage('Please login first!', true);
    return;
  }
  submitting.value = true;
  try {
    if (selectedStudent.value) {
      await axios.put(`/api/students/${selectedStudent.value.id}`, formData);
      showMessage('Student updated successfully!');
      selectedStudent.value = null;
    } else {
      await axios.post('/api/students', formData);
      showMessage('Student created successfully!');
    }
    await fetchStudents();
  } catch (err) {
    showMessage(err.response?.data?.message || 'Operation failed. Admin role required.', true);
  } finally {
    submitting.value = false;
  }
};

const handleDelete = async (id) => {
  if (!confirm('Are you sure you want to delete this student?')) return;
  loading.value = true;
  try {
    await axios.delete(`/api/students/${id}`);
    showMessage('Student deleted successfully!');
    await fetchStudents();
  } catch (err) {
    showMessage(err.response?.data?.message || 'Delete failed. Admin role required.', true);
  } finally {
    loading.value = false;
  }
};

const setEditing = (student) => {
  selectedStudent.value = student;
};
</script>