<script setup>
import { ref } from 'vue';
import * as XLSX from 'xlsx';

const excelData = ref([]);
const headers = ref([]);
const columnMapping = ref({}); // To store column mapping
const selectedFormat = ref('xlsx'); // Default export format

const customTemplates = ref([]);
const newTemplateName = ref('');
const newTemplateHeadersInput = ref('');
const selectedTemplate = ref('audio-import'); // 'audio-import' or name of custom template

// Fixed template headers
const fixedTemplateHeaders = [
  'product-title', 'product-clean-title', 'product-title-version', 'product-artists-and-roles',
  'product-label', 'product-catid', 'product-upc', 'product-digital-street',
  'product-physical-street', 'product-original-street', 'product-explicit', 'product-genre1',
  'product-genre2', 'product-c-line', 'product-p-line', 'product-notes', 'product-art-path',
  'product-territory-offers', 'product-rightsholder', 'product-copyright', 'asset-title',
  'asset-clean-title', 'asset-title-version', 'asset-c-line', 'asset-c-year', 'asset-p-line',
  'asset-p-year', 'asset-disc-number', 'asset-track-sequence', 'asset-artist-and-roles',
  'asset-isrc', 'asset-duration', 'asset-preview-start', 'asset-explicit', 'asset-genre1',
  'asset-genre2', 'asset-individual-trk-sale', 'asset-filepath', 'asset-rightsholder',
  'asset-copyright', 'asset-priority', 'asset-notes', 'asset-master-rights', 'contract_name',
  'contract-territories-allowed', 'contract-territories-denied', 'work-title', 'work-iswc',
  'work-title-code', 'work-pro', 'work-lyrics', 'work-writer', 'work-rights', 'work-publisher',
  'work-copyright', 'work-filepath', 'sunrise', 'product-type', 'asset-customid',
  'work-youtubecustomid', 'asset-youtubecustomid'
];

const templateHeaders = computed(() => {
  if (selectedTemplate.value === 'fixed') {
    return fixedTemplateHeaders;
  } else {
    const custom = customTemplates.value.find(t => t.name === selectedTemplate.value);
    return custom ? custom.headers : [];
  }
});

const loadCustomTemplates = () => {
  const storedTemplates = localStorage.getItem('customExcelTemplates');
  if (storedTemplates) {
    customTemplates.value = JSON.parse(storedTemplates);
  }
};

const saveCustomTemplate = () => {
  if (newTemplateName.value && newTemplateHeadersInput.value) {
    const headersArray = newTemplateHeadersInput.value.split(',').map(h => h.trim()).filter(h => h);
    if (headersArray.length > 0) {
      // Check if template name already exists for editing
      const existingIndex = customTemplates.value.findIndex(t => t.name === newTemplateName.value);
      if (existingIndex !== -1) {
        // Update existing template
        customTemplates.value[existingIndex].headers = headersArray;
        alert('Template updated successfully!');
      } else {
        // Add new template
        customTemplates.value.push({
          name: newTemplateName.value,
          headers: headersArray
        });
        alert('Template saved successfully!');
      }
      localStorage.setItem('customExcelTemplates', JSON.stringify(customTemplates.value));
      newTemplateName.value = '';
      newTemplateHeadersInput.value = '';
      // Automatically select the newly saved/updated template
      selectedTemplate.value = newTemplateName.value;
      resetMapping();
    } else {
      alert('Please enter valid headers for the template.');
    }
  } else {
    alert('Please enter both template name and headers.');
  }
};

const editCustomTemplate = (template) => {
  newTemplateName.value = template.name;
  newTemplateHeadersInput.value = template.headers.join(', ');
};

const deleteCustomTemplate = (templateName) => {
  if (confirm(`Are you sure you want to delete the template "${templateName}"?`)) {
    customTemplates.value = customTemplates.value.filter(t => t.name !== templateName);
    localStorage.setItem('customExcelTemplates', JSON.stringify(customTemplates.value));
    if (selectedTemplate.value === templateName) {
      selectedTemplate.value = 'audio-import'; // Fallback to fixed template if deleted
      resetMapping();
    }
    alert('Template deleted successfully!');
  }
};

// Load templates on component mount
import { onMounted, computed, watch } from 'vue';
onMounted(() => {
  loadCustomTemplates();
});

// Watch for changes in selectedTemplate and reset mapping
watch(selectedTemplate, () => {
  resetMapping();
});

const handleFileUpload = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = (e) => {
      const data = new Uint8Array(e.target.result);
      const workbook = XLSX.read(data, { type: 'array' });
      const firstSheetName = workbook.SheetNames[0];
      const worksheet = workbook.Sheets[firstSheetName];
      const jsonData = XLSX.utils.sheet_to_json(worksheet, { header: 1 });
      
      if (jsonData.length > 0) {
        headers.value = jsonData[0];
        excelData.value = jsonData.slice(1);
        resetMapping(); // Inicializa el mapeo después de cargar las cabeceras
      }
    };
    reader.readAsArrayBuffer(file);
  }
};

const downloadProcessedFile = () => {
  const processedData = excelData.value.map(row => {
    const newRow = [];
    templateHeaders.value.forEach(templateCol => {
      const mapping = columnMapping.value[templateCol];
      let value = '';

      if (mapping && mapping.source) {
        const sourceIndex = headers.value.indexOf(mapping.source);
        if (sourceIndex !== -1) {
          value = row[sourceIndex];
        }
      } else if (mapping && mapping.defaultValue) {
        value = mapping.defaultValue;
      }

      // Formatear fechas si la columna es una de las columnas de fecha esperadas
      if (dateColumns.includes(templateCol)) {
        value = excelSerialDateToJSDate(value);
      }
      newRow.push(value);
    });
    return newRow;
  });

  // Create a new worksheet with processed data
  const newWorksheet = XLSX.utils.aoa_to_sheet([templateHeaders.value, ...processedData]);
  const newWorkbook = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(newWorkbook, newWorksheet, 'Sheet1');

  // Descargar el nuevo archivo
  if (selectedFormat.value === 'xlsx') {
    XLSX.writeFile(newWorkbook, 'processed_data.xlsx');
  } else if (selectedFormat.value === 'csv') {
    const csv = XLSX.utils.sheet_to_csv(newWorksheet);
    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.setAttribute('download', 'processed_data.csv');
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  }
};
</script>

<template>
  <div class="container py-4">
    <div class="card shadow-lg p-4 p-md-5">
      <h1 class="text-center mb-4">Excel Formatting Assistant</h1>

      <div class="border border-dashed border-secondary rounded p-4 text-center mb-4">
        <p class="text-muted mb-2">Drag and drop your Excel file here or click to select</p>
        <input type="file" @change="handleFileUpload" class="d-none" id="file-upload" />
        <label for="file-upload" class="btn btn-primary">
          Select File
        </label>
      </div>

      <div class="mb-4">
        <h2 class="mb-3">Template Management</h2>
        <div class="row g-3 mb-3">
          <div class="col-md-6">
            <label for="newTemplateName" class="form-label">New Template Name:</label>
            <input type="text" id="newTemplateName" v-model="newTemplateName" class="form-control" placeholder="e.g., My Custom Template" />
          </div>
          <div class="col-md-6">
            <label for="newTemplateHeaders" class="form-label">New Template Headers (comma-separated):</label>
            <input type="text" id="newTemplateHeaders" v-model="newTemplateHeadersInput" class="form-control" placeholder="header1, header2, header3" />
          </div>
        </div>
        <button @click="saveCustomTemplate" class="btn btn-secondary mb-4">Save Custom Template</button>

        <h3 class="mb-2">Select Template:</h3>
        <div class="d-flex align-items-center mb-3">
          <select v-model="selectedTemplate" @change="resetMapping" class="form-select me-2">
            <option value="audio-import">Audio Import</option>
            <option v-for="template in customTemplates" :key="template.name" :value="template.name">{{ template.name }}</option>
          </select>
          <button
            v-if="selectedTemplate !== 'audio-import'"
            @click="editCustomTemplate(customTemplates.find(t => t.name === selectedTemplate))"
            class="btn btn-info btn-sm me-2"
          >
            Edit
          </button>
          <button
            v-if="selectedTemplate !== 'audio-import'"
            @click="deleteCustomTemplate(selectedTemplate)"
            class="btn btn-danger btn-sm"
          >
            Delete
          </button>
        </div>
      </div>

      <div v-if="excelData.length > 0" class="mt-4">
        <h2 class="mb-3">Column Mapping</h2>
        <div class="row g-3 mb-4">
          <div v-for="templateCol in templateHeaders" :key="templateCol" class="col-md-4">
            <label :for="templateCol" class="form-label">{{ templateCol }}</label>
            <select
              :id="templateCol"
              v-model="columnMapping[templateCol].source"
              class="form-select mb-2"
            >
              <option value="">-- Select Column --</option>
              <option v-for="header in headers" :key="header" :value="header">{{ header }}</option>
            </select>
            <input
              type="text"
              v-model="columnMapping[templateCol].defaultValue"
              class="form-control"
              placeholder="Default Value (optional)"
            />
          </div>
        </div>

        <div class="d-flex justify-content-between align-items-center mb-4">
          <h2 class="mb-0">Data Preview</h2>
          <div class="d-flex align-items-center">
            <div class="me-3">
              <label for="export-format" class="form-label mb-0">Export Format:</label>
              <select
                id="export-format"
                v-model="selectedFormat"
                class="form-select form-select-sm"
              >
                <option value="xlsx">Excel (xlsx)</option>
                <option value="csv">CSV (csv)</option>
              </select>
            </div>
            <button @click="downloadProcessedFile" class="btn btn-success">
              Download Processed File
            </button>
          </div>
        </div>
        <div class="table-responsive mb-4">
          <table class="table table-bordered table-hover">
            <thead>
              <tr>
                <th v-for="header in headers" :key="header" class="bg-light text-start">{{ header }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, rowIndex) in excelData" :key="rowIndex">
                <td v-for="(cell, cellIndex) in row" :key="cellIndex">{{ cell }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>