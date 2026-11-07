<template>
  <div class="p-4 bg-white rounded border shadow-sm">
    <h3 class="text-xl font-bold mb-3">{{ isEditing ? 'Edit Student' : 'Add New Student' }}</h3>
    
    <!-- Role Restriction Notice for Regular Users -->
    <div v-if="userRole === 'user'" class="p-3 bg-amber-50 text-amber-800 text-sm rounded border border-amber-200">
      🔒 Logged in as <strong>Regular User</strong>. Only Administrators can add or modify student records.
    </div>

    <!-- Form for Administrators -->
    <form v-else @submit.prevent="handleSubmit" class="space-y-3">
      <div>
        <label class="block text-sm font-medium text-gray-700">Name</label>
        <input 
          v-model="form.name" 
          type="text" 
          required 
          :disabled="submitting"
          class="w-full p-2 border rounded mt-1 disabled:bg-gray-100"
        />
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700">Email</label>
        <input 
          v-model="form.email" 
          type="email" 
          required 
          :disabled="submitting"
          class="w-full p-2 border rounded mt-1 disabled:bg-gray-100"
        />
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700">Course</label>
        <input 
          v-model="form.course" 
          type="text" 
          required 
          :disabled="submitting"
          class="w-full p-2 border rounded mt-1 disabled:bg-gray-100"
        />
      </div>

      <div class="flex gap-2 pt-2">
        <button 
          type="submit" 
          :disabled="submitting"
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 w-full disabled:opacity-50"
        >
          <span v-if="submitting">Processing...</span>
          <span v-else>{{ isEditing ? 'Update Student' : 'Add Student' }}</span>
        </button>
        <button 
          v-if="isEditing" 
          type="button" 
          @click="$emit('cancel-edit')" 
          class="px-4 py-2 bg-gray-500 text-white rounded hover:bg-gray-600"
        >
          Cancel
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { reactive, computed, watch } from 'vue';

const props = defineProps({
  editingStudent: {
    type: Object,
    default: null
  },
  userRole: {
    type: String,
    default: ''
  },
  submitting: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['save-student', 'cancel-edit']);

// Managed state using reactive()
const form = reactive({
  name: '',
  email: '',
  course: ''
});

const isEditing = computed(() => !!props.editingStudent);

watch(() => props.editingStudent, (newVal) => {
  if (newVal) {
    form.name = newVal.name;
    form.email = newVal.email;
    form.course = newVal.course;
  } else {
    form.name = '';
    form.email = '';
    form.course = '';
  }
}, { immediate: true });

const handleSubmit = () => {
  emit('save-student', { ...form });
  if (!isEditing.value) {
    form.name = '';
    form.email = '';
    form.course = '';
  }
};
</script>