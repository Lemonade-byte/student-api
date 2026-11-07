<template>
  <div class="p-4 bg-white rounded border shadow-sm">
    <h3 class="text-xl font-bold mb-3">{{ isEditing ? 'Edit Student' : 'Add New Student' }}</h3>
    
    <form @submit.prevent="handleSubmit" class="space-y-3">
      <div>
        <label class="block text-sm font-medium text-gray-700">Name</label>
        <input 
          v-model="form.name" 
          type="text" 
          required 
          class="w-full p-2 border rounded mt-1"
        />
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700">Email</label>
        <input 
          v-model="form.email" 
          type="email" 
          required 
          class="w-full p-2 border rounded mt-1"
        />
      </div>

      <div>
        <label class="block text-sm font-medium text-gray-700">Course</label>
        <input 
          v-model="form.course" 
          type="text" 
          required 
          class="w-full p-2 border rounded mt-1"
        />
      </div>

      <div class="flex gap-2 pt-2">
        <button 
          type="submit" 
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 w-full"
        >
          {{ isEditing ? 'Update Student' : 'Add Student' }}
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

<script>
export default {
  name: 'StudentForm',
  props: {
    editingStudent: {
      type: Object,
      default: null
    }
  },
  data() {
    return {
      form: {
        name: '',
        email: '',
        course: ''
      }
    }
  },
  computed: {
    isEditing() {
      return !!this.editingStudent;
    }
  },
  watch: {
    editingStudent: {
      immediate: true,
      handler(newVal) {
        if (newVal) {
          this.form = { ...newVal };
        } else {
          this.form = { name: '', email: '', course: '' };
        }
      }
    }
  },
  methods: {
    handleSubmit() {
      this.$emit('save-student', this.form);
      if (!this.isEditing) {
        this.form = { name: '', email: '', course: '' };
      }
    }
  }
}
</script>