<script setup>
import { ref, onMounted, computed } from 'vue';
import * as XLSX from 'xlsx';

import InfoSection from './components/InfoSection.vue';
import FileUploadSection from './components/FileUploadSection.vue';
import ExportOptionsSection from './components/ExportOptionsSection.vue';
import ColumnMappingSection from './components/ColumnMappingSection.vue';

const excelData = ref([]);
const headers = ref([]);
const isLoading = ref(false);
const selectedFormat = ref('xlsx'); // Default export format

// Fixed template headers (Audio Import)
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

// Initialize columnMapping with default structure for all fixedTemplateHeaders
const initialColumnMapping = {};
fixedTemplateHeaders.forEach(templateCol => {
  initialColumnMapping[templateCol] = {
    source: '',
    defaultValue: ''
  };
});
const columnMapping = ref(initialColumnMapping);

const templateHeaders = computed(() => fixedTemplateHeaders); // Always use fixed template headers

// Column names that are expected to contain dates and need formatting
const dateColumns = [
  'product-digital-street',
  'product-physical-street',
  'product-original-street',
  'sunrise'
];

// Function to convert Excel serial date to readable date (YYYY-MM-DD)
const excelSerialDateToJSDate = (serial) => {
  if (typeof serial !== 'number' || isNaN(serial)) {
    return '';
  }
  const date = new Date(Math.round((serial - 25569) * 86400 * 1000));
  return date.toISOString().split('T')[0];
};

const resetMapping = () => {
  // Only update sources based on new headers, keep default values if set
  templateHeaders.value.forEach(templateCol => {
    if (headers.value.includes(templateCol)) {
      columnMapping.value[templateCol].source = templateCol;
    } else {
      columnMapping.value[templateCol].source = ''; // Clear source if not found
    }
  });
};

onMounted(() => {
  resetMapping(); // Initialize mapping on component mount
});

const handleFileUpload = (file) => {
  if (file) {
    isLoading.value = true; // Set loading to true
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
        resetMapping(); // Re-initialize mapping after loading new headers
      }
      isLoading.value = false; // Set loading to false after processing
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

      // Format dates if the column is one of the expected date columns
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

  // Download the new file
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

const updateColumnMapping = (newMapping) => {
  columnMapping.value = newMapping;
};
</script>

<template>
  <div class="container py-4">
    <div class="card shadow-lg p-4 p-md-5 my-4">
      <img src="/public/logo.png" width="250px" class="m-auto pb-4" alt="Excel Formatting Assistant">
      <InfoSection />

      <FileUploadSection :isLoading="isLoading" @file-uploaded="handleFileUpload" />

      <!-- Export Options (moved for accessibility) -->
      <ExportOptionsSection
        :selectedFormat="selectedFormat"
        @update:selectedFormat="selectedFormat = $event"
        @download-file="downloadProcessedFile"
      />

      <!-- Column Mapping Section (always visible) -->
      <ColumnMappingSection
        :templateHeaders="templateHeaders"
        :availableHeaders="headers"
        :columnMapping="columnMapping"
        @update:columnMapping="updateColumnMapping"
      />

      <!-- Data Preview Section (only visible if excelData is available) -->
      <div v-if="excelData.length > 0" class="mt-4 pt-4 border-top">
        <h2 class="mb-3">Data Preview</h2>
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