<template>
  <div class="d-flex justify-content-center align-items-center mb-4">
    <div class="me-3">
      <label for="export-format" class="form-label mb-0">Export Format:</label>
      <select
        id="export-format"
        :value="selectedFormat"
        @change="$emit('update:selectedFormat', $event.target.value)"
        class="form-select form-select-sm"
      >
        <option value="xlsx">Excel (xlsx)</option>
        <option value="csv">CSV (csv)</option>
      </select>
    </div>
    <button @click="$emit('download-file')" class="btn btn-success">
      Download Processed File
    </button>
  </div>
</template>

<script setup>
defineProps({
  selectedFormat: {
    type: String,
    required: true,
  },
});

defineEmits(['update:selectedFormat', 'download-file']);
</script>

<style scoped>
/* Component-specific styles if any */
</style>