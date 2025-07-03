<template>
  <div class="modal fade" id="contentbridgeTutorialModal" tabindex="-1" aria-labelledby="contentbridgeTutorialModalLabel" aria-hidden="true">
    <div class="modal-dialog modal-lg modal-dialog-scrollable">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="contentbridgeTutorialModalLabel">Contentbridge Import Tutorials</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
          <h4>Tutorial 1: How to Import Audio Files</h4>
          <p>This section will contain detailed steps and screenshots on how to import audio files into Contentbridge using the generated Excel template.</p>
          <ul>
            <li>Step 1: Prepare your data...</li>
            <li>Step 2: Map columns in the Excel Formatting Assistant...</li>
            <li>Step 3: Download the processed file...</li>
            <li>Step 4: Upload to Contentbridge...</li>
          </ul>
          <hr>
          <h4>Tutorial 2: How to Import Music Video Files</h4>
          <p>This section will cover the process of importing music video files, highlighting any differences from audio file imports.</p>
          <ul>
            <li>Step 1: Prepare your video data...</li>
            <li>Step 2: Use the Music Video Import template...</li>
            <li>Step 3: Ensure video paths are correct...</li>
            <li>Step 4: Finalize import in Contentbridge...</li>
          </ul>
          <hr>
          <h4>General Tips:</h4>
          <ul>
            <li>Always double-check your mapped columns.</li>
            <li>Use the default value option for consistent data.</li>
            <li>Refer to Contentbridge documentation for specific field requirements.</li>
          </ul>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
// No specific script logic needed for this static modal component
</script>

<style scoped>
/* Add any component-specific styles here if needed */
</style>