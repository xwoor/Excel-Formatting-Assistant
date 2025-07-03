<template>
  <div class="mt-4 pt-4 border-top">
    <h2 class="mb-3">Column Mapping</h2>
    <div class="accordion" id="columnMappingAccordion">
      <div class="accordion-item" v-for="(headers, groupName) in groupedTemplateHeaders" :key="groupName">
        <h2 class="accordion-header" :id="`heading${groupName.replace(/\s/g, '')}`">
          <button
            class="accordion-button collapsed"
            type="button"
            data-bs-toggle="collapse"
            :data-bs-target="`#collapse${groupName.replace(/\s/g, '')}`"
            aria-expanded="false"
            :aria-controls="`collapse${groupName.replace(/\s/g, '')}`"
          >
            {{ groupName }}
          </button>
        </h2>
        <div
          :id="`collapse${groupName.replace(/\s/g, '')}`"
          class="accordion-collapse collapse"
          :aria-labelledby="`heading${groupName.replace(/\s/g, '')}`"
          data-bs-parent="#columnMappingAccordion"
        >
          <div class="accordion-body">
            <div class="row g-3 mb-4">
              <div v-for="templateCol in headers" :key="templateCol" class="col-md-6">
                <label :for="templateCol" class="form-label">{{ templateCol }}</label>
                <select
                  :id="templateCol"
                  :value="columnMapping[templateCol] ? columnMapping[templateCol].source : ''"
                  @change="updateMappingSource(templateCol, $event.target.value)"
                  class="form-select mb-2"
                >
                  <option value="">-- Select Column --</option>
                  <option v-for="header in availableHeaders" :key="header" :value="header">{{ header }}</option>
                </select>
                <input
                  type="text"
                  :value="columnMapping[templateCol] ? columnMapping[templateCol].defaultValue : ''"
                  @input="updateMappingDefaultValue(templateCol, $event.target.value)"
                  class="form-control"
                  placeholder="Default Value (optional)"
                />
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  templateHeaders: {
    type: Array,
    required: true,
  },
  availableHeaders: {
    type: Array,
    required: true,
  },
  columnMapping: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(['update:columnMapping']);

const groupedTemplateHeaders = computed(() => {
  const groups = {};
  props.templateHeaders.forEach(header => {
    let prefix = 'Other';
    if (header.startsWith('product-')) prefix = 'Product Fields';
    else if (header.startsWith('asset-')) prefix = 'Asset Fields';
    else if (header.startsWith('work-')) prefix = 'Work Fields';
    else if (header.startsWith('contract-')) prefix = 'Contract Fields';

    if (!groups[prefix]) {
      groups[prefix] = [];
    }
    groups[prefix].push(header);
  });
  return groups;
});

const updateMappingSource = (templateCol, value) => {
  const newMapping = { ...props.columnMapping };
  if (!newMapping[templateCol]) {
    newMapping[templateCol] = { source: '', defaultValue: '' };
  }
  newMapping[templateCol].source = value;
  emit('update:columnMapping', newMapping);
};

const updateMappingDefaultValue = (templateCol, value) => {
  const newMapping = { ...props.columnMapping };
  if (!newMapping[templateCol]) {
    newMapping[templateCol] = { source: '', defaultValue: '' };
  }
  newMapping[templateCol].defaultValue = value;
  emit('update:columnMapping', newMapping);
};
</script>

<style scoped>
/* Component-specific styles if any */
</style>