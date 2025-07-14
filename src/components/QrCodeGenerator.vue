<template>
  <div class="qr-code-generator">
    <div class="readme">
      <h2>QR Code Generator</h2>
      <p>
        Enter your text below to generate a QR code. Only alphanumeric characters and spaces are allowed.
      </p>
    </div>
    <v-text-field
      v-model="inputText"
      label="Enter text to generate QR code"
      outlined
      clearable
      @keyup.enter="generateQrCode"
      class="qr-input"
      :error="!isValid && inputText.length > 0"
      :error-messages="!isValid && inputText.length > 0 ? ['Only alphanumeric characters and spaces are allowed.'] : []"
    />
    <v-btn color="primary" @click="generateQrCode" :disabled="!isValid || !inputText">Generate QR Code</v-btn>
    <div v-if="qrCodeUrl" class="qr-code-image">
      <img :src="qrCodeUrl" alt="Generated QR Code" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue';
import QRCode from 'qrcode';

const inputText = ref('');
const qrCodeUrl = ref<string | null>(null);

// Regex: allow only alphanumeric and spaces
const inputRegex = /^[a-zA-Z0-9 ]*$/;
const isValid = computed(() => inputRegex.test(inputText.value));

const generateQrCode = async () => {
  if (!inputText.value || !isValid.value) {
    qrCodeUrl.value = null;
    return;
  }
  try {
    qrCodeUrl.value = await QRCode.toDataURL(inputText.value, {
      width: 300,
      margin: 2,
      color: {
        dark: '#000000',
        light: '#ffffff',
      },
    });
  } catch (error) {
    qrCodeUrl.value = null;
    // Optionally, handle error (e.g., show a message)
  }
};
</script>

<style scoped>
.qr-code-generator {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  max-width: 400px;
  margin: 0 auto;
}
.readme {
  text-align: center;
  margin-bottom: 1rem;
}
.qr-input {
  width: 100%;
}
.qr-code-image img {
  width: 300px;
  height: 300px;
  border: 1px solid #eee;
  background: #fff;
}
</style> 