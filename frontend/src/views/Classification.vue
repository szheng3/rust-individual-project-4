<script setup>
import {ref} from "vue";

const selectedFile = ref(null);

const onFileInputChange = (event) => {
  selectedFile.value = event.target.files[0];
}

const upload = async () => {
  const formData = new FormData();
  formData.append('image', selectedFile.value);

  try {
    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData,
    });

    if (response.ok) {
      console.log('Image uploaded successfully');
    } else {
      console.error('Failed to upload image');
    }
  } catch (error) {
    console.error(error);
  }
}
</script>
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <input type="file" ref="fileInput" @change="onFileInputChange"/>
    <button @click="upload">Upload</button>
  </div>
</template>

<style>
@media (min-width: 1024px) {
  .about {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
