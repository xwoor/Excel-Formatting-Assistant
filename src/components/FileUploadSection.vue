<template>
  <div class="border border-dashed border-secondary rounded p-4 text-center my-4">
    <p class="text-muted mb-3">Drag and drop your Excel file here or click to select</p>
    <input type="file" @change="handleFileUpload" class="d-none" id="file-upload" />
    <label for="file-upload" class="btn btn-primary">
      Select File
    </label>
    <div v-if="isLoading" class="spinner-border text-primary ms-2" role="status">
      <span class="visually-hidden">Loading...</span>
    </div>
  </div>
</template>

<script setup>
/**
 * @typedef {Object} FileUploadSectionProps
 * @property {boolean} isLoading - Indicates if a file is currently being processed or loaded.
 */

/**
 * @property {FileUploadSectionProps} props
 */
defineProps({
  isLoading: {
    type: Boolean,
    required: true,
  },
});

const emit = defineEmits(['file-uploaded']);

/**
 * Handles the file upload event.
 * Emits a 'file-uploaded' event with the selected file.
 * @param {Event} event - The DOM event from the file input.
 * @emits {File} file-uploaded - The selected file object.
 */
const handleFileUpload = (event) => {
  emit('file-uploaded', event.target.files[0]);
};
</script>

<style scoped>
/* Component-specific styles if any */
</style>