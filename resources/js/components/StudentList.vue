<template>
  <div class="mt-4">
    <h3 class="text-xl font-bold mb-2">Student List</h3>
    
    <!-- Loading State -->
    <div v-if="loading" class="text-blue-600 font-medium p-4 bg-white rounded border animate-pulse">
      Loading student records...
    </div>

    <!-- Empty State -->
    <div v-else-if="students.length === 0" class="text-gray-500 italic p-4 bg-white rounded border">
      No student records found.
    </div>

    <!-- Student Items -->
    <div v-else class="space-y-2">
      <div 
        v-for="student in students" 
        :key="student.id" 
        class="p-4 border rounded shadow-sm flex justify-between items-center bg-white"
      >
        <div>
          <p class="font-semibold text-lg text-gray-800">{{ student.name }}</p>
          <p class="text-sm text-gray-600">{{ student.email }} | Course: {{ student.course }}</p>
        </div>

        <!-- Role-Based Restriction: Show Edit/Delete only to Administrator -->
        <div v-if="userRole === 'admin'" class="space-x-2">
          <button 
            @click="$emit('edit-student', student)" 
            class="px-3 py-1 bg-yellow-500 text-white rounded text-sm hover:bg-yellow-600"
          >
            Edit
          </button>
          <button 
            @click="$emit('delete-student', student.id)" 
            class="px-3 py-1 bg-red-600 text-white rounded text-sm hover:bg-red-700"
          >
            Delete
          </button>
        </div>

        <!-- Read-Only Tag for Regular Users -->
        <div v-else-if="userRole === 'user'" class="text-xs text-gray-400 italic">
          Read-only view
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  students: {
    type: Array,
    required: true
  },
  userRole: {
    type: String,
    default: ''
  },
  loading: {
    type: Boolean,
    default: false
  }
});

defineEmits(['edit-student', 'delete-student']);
</script>