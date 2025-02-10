
<script>
  import { uploadMapImage, getMapImageUrl } from '$lib/storage';
  
  let selectedFile;
  let uploadStatus = '';
  let currentMapUrl = '';
  let uploadProgress = 0;

  async function loadCurrentImage() {
    try {
      currentMapUrl = await getMapImageUrl();
    } catch (error) {
      console.error('Error loading image:', error);
      uploadStatus = `Error loading image: ${error.message}`;
    }
  }

  loadCurrentImage();

  async function handleFileSelect(event) {
    selectedFile = event.target.files[0];
    if (selectedFile && selectedFile.size > 5 * 1024 * 1024) {
      uploadStatus = 'Error: File size must be under 5MB';
      selectedFile = null;
      return;
    }
    uploadStatus = '';
    uploadProgress = 0;
  }

  async function handleUpload() {
    if (!selectedFile) return;
    
    try {
      uploadStatus = 'Uploading...';
      uploadProgress = 10;
      
      console.log('Starting upload with file:', {
        name: selectedFile.name,
        size: selectedFile.size,
        type: selectedFile.type
      });

      const result = await uploadMapImage(selectedFile);
      console.log('Upload result:', result);
      
      uploadStatus = 'Upload successful!';
      uploadProgress = 100;
      
      // Reload the image
      currentMapUrl = await getMapImageUrl() + '?t=' + Date.now();
      
      // Reset file input
      selectedFile = null;
      
    } catch (error) {
      console.error('Upload error:', error);
      uploadStatus = `Upload failed: ${error.message}`;
      uploadProgress = 0;
    }
  }
</script>

<div class="p-4 border rounded-lg bg-gray-800">
  <h2 class="text-xl font-bold mb-4">Mansion Map Upload</h2>
  
  <div class="mb-4">
    <input 
      type="file" 
      accept="image/jpeg,image/png"
      on:change={handleFileSelect}
      class="mb-2">
    
    <button 
      on:click={handleUpload}
      disabled={!selectedFile}
      class="bg-blue-500 text-white px-4 py-2 rounded disabled:opacity-50">
      Upload Map Image
    </button>
  </div>

  {#if uploadStatus}
    <p class="mb-4">{uploadStatus}</p>
  {/if}

  {#if uploadProgress > 0}
    <div class="w-full bg-gray-700 rounded-full h-2.5 mb-4">
      <div 
        class="bg-blue-500 h-2.5 rounded-full transition-all duration-300" 
        style="width: {uploadProgress}%">
      </div>
    </div>
  {/if}

  <div class="mt-4">
    <h3 class="font-bold mb-2">Current Map Image:</h3>
    {#if currentMapUrl}
      <img 
        src={currentMapUrl} 
        alt="Mansion Map" 
        class="max-w-xl rounded-lg shadow-lg"
        on:error={(e) => {
          console.error('Image load error:', e);
          currentMapUrl = '';
        }} 
      />
    {/if}
  </div>
</div>
</boltArtifact>
```