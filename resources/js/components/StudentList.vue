<template>
  <div class="mt-4">
    <h3 class="text-xl font-bold mb-2">Student List</h3>
    
    <!-- Conditional content using v-if -->
    <div v-if="students.length === 0" class="text-gray-500 italic p-4 bg-white rounded border">
      No student records found.
    </div>

    <!-- Displaying records using v-for -->
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
        <div class="space-x-2">
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
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StudentList',
  props: {
    students: {
      type: Array,
      required: true
    }
  }
}
</script>