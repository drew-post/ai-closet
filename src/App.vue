<template>
  <div class="container">
    <h1>Outfit Suggestions</h1>
    <div class="prompt-input">
      <label for="prompt">I want an outfit for:</label>
      <input type="text" id="prompt" v-model="prompt" />
      <button @click="getOutfitAdvice">Get Outfit Advice</button>
    </div>
    <div v-if="adviceLoading" class="loading">
      <p>Loading...</p>
    </div>
    <div v-if="advice && !adviceLoading" class="response">
      <div class="advice">
        <h2>Outfit Advice:</h2>
        <p>{{ advice }}</p>
      </div>
      <div class="picture-gen-prompt">
        <label for="picture-button">Want to see a picture of this outfit?</label>
        <button id="picture-button" @click="getOutfitPicture">Generate</button>
      </div>
      <div v-if="pictureLoading" class="loading">
        <p>Loading...</p>
      </div>
      <div v-if="picture && !pictureLoading" class="generated-picture">
        <img :src="picture" width="512" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  components: {
  },
  data() {
    return {
      prompt: '',
      advice: null,
      picture: null,
      adviceLoading: false,
      pictureLoading: false,
    };
  },
  methods: {
    async getOutfitAdvice() {
      if (!this.prompt) return;

      this.adviceLoading = true; 
      this.picture = null;

      // Example few-shot learning prompts and responses
      const fewShotExamples = [
        "Scenario:\nWinter\nOutfit Suggestion:\nWear a cozy, black sweater with a light weight turtleneck underneath for warmth, pair that with some dark wash jeans. For outerwear, go with a warm, neutral colored wool overcoat, and for shoes, try some knee high boots. Don't forget a scarf and gloves!\n",
        "Scenario:\nJob Interview\nOutfit Suggestion:\nOpt for a well-fitted pantsuit or a blazer with a matching dress skirt. Stick to neutral colors like black, navy, or charcoal gray. Ensure the fit is comfortable and professional. Choose closed-toe, low-heeled pumps or dress shoes in a neutral color. Make sure they're clean and polished.\n",
        "Scenario:\nBeach Vacation\nOutfit Suggestion:\nGo for light and breezy clothing like a sarong, sundress, or a beach tunic, or go for comfortable shorts and a tank top. Light pastel colors like soft blues, pale pinks, mint green, and lavender are refreshing and evoke a sense of tranquility, which can be perfect for a beach day. Don't forget sunscreen!\n",
      ];

      try {
        const apiKey = process.env.VUE_APP_OPENAI_API_KEY;
        const apiUrl = 'https://api.openai.com/v1/engines/davinci/completions';
        const promptWithFewShotExamples = `${fewShotExamples.join("\n")}\nYou want an outfit for a scenario where you're ${this.prompt}. What should you wear? Make it relevant to 1990s fashion trends`;
        let response = await this.sendAPIRequest(apiUrl, apiKey, promptWithFewShotExamples);
        response = response.data.choices[0].text;
        response = response.split(/\r?\n/);
        this.advice = response[5];
      } catch (error) {
        console.error('Error getting outfit advice:', error);
      } finally {
        this.adviceLoading = false; 
      }
    },
    async getOutfitPicture() {
      if (!this.prompt) return;

      this.pictureLoading = true;

      try {
        const apiKey = process.env.VUE_APP_OPENAI_API_KEY;
        const apiUrl = 'https://api.openai.com/v1/images/generations';
        const picturePrompt = `Generate a picture that displays a flat lay view of this outfit: ${this.advice}. Picture is based on 90s fashion`;
        let response = await this.sendPictureAPIRequest(apiUrl, apiKey, picturePrompt);
        console.log(response);
        this.picture = response.data.data[0].url;
      } catch (error) {
        console.error('Error getting outfit advice:', error);
      } finally {
        this.pictureLoading = false; 
      }
    },
    async sendAPIRequest(url, apiKey, prompt) {
      const data = {
        prompt: prompt,
        max_tokens: 200,
        temperature: 0.5,
      };
      const config = {
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json',
        },
      };
      const response = await axios.post(url, data, config);
      return response;
    },
    async sendPictureAPIRequest(url, apiKey, prompt) {
      const data = {
        prompt: prompt,
        n: 1,
        size: "1024x1024"
      };
      const config = {
        headers: {
          'Authorization': `Bearer ${apiKey}`,
          'Content-Type': 'application/json',
        },
      };
      const response = await axios.post(url, data, config);
      return response;
    },
  },
};
</script>
<style>
.container {
  display: flex;
  flex-direction: column;
  gap: 35px;
  font-size: 20px;
  padding: 50px;
}
.prompt-input {
  display: flex;
  flex-direction: row;
  gap: 10px;
}
.response {
  display: flex;
  flex-direction: column;
  gap: 20px;
}
.advice {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.picture-gen-prompt {
  display: flex;
  flex-direction: row;
  gap: 10px;
}
</style>
