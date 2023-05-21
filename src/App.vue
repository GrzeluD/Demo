<template>
  <div>
    <input type="file" ref="fileInput" @change="handleFileChange">
    <button @click="analyzeImage">Analizuj obraz</button>

    <ul>
      <li v-for="label in labels" :key="label">{{ label }}</li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      imageData: null,
      labels: []
    };
  },
  methods: {
    handleFileChange(event) {
      const file = event.target.files[0];
      const reader = new FileReader();

      reader.onload = (event) => {
        this.imageData = event.target.result;
      };

      reader.readAsDataURL(file);
    },
    async analyzeImage() {
      try {
        const response = await axios.post(
            `https://vision.googleapis.com/v1/images:annotate?key=${process.env.GOOGLE_VISION_API_KEY}`,
            {
              requests: [
                {
                  image: {
                    content: this.imageData.substring(this.imageData.indexOf(',') + 1)
                  },
                  features: [
                    {
                      type: 'LABEL_DETECTION',
                      maxResults: 5
                    }
                  ]
                }
              ]
            }
        );

        this.labels = response.data.responses[0].labelAnnotations.map(label => label.description);
      } catch (error) {
        console.error(error);
      }
    }
  }
};
</script>