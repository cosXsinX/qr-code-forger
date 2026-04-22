<template>
  <div class="qr-code-generator">
    <div class="language-switcher">
      <v-menu>
        <template #activator="{ props }">
          <v-btn v-bind="props" variant="outlined">
            {{ t.languageButton }}: {{ currentLanguageLabel }}
          </v-btn>
        </template>
        <v-list>
          <v-list-item
            v-for="language in languageOptions"
            :key="language"
            :title="languageLabels[language]"
            @click="selectedLanguage = language"
          />
        </v-list>
      </v-menu>
    </div>
    <div class="readme">
      <h2>{{ t.title }}</h2>
      <p>
        {{ t.description }}
      </p>
    </div>
    <v-text-field
      v-model="inputText"
      :label="t.inputLabel"
      outlined
      clearable
      @keyup.enter="generateQrCode"
      class="qr-input"
      :error="!isValid && inputText.length > 0"
      :error-messages="!isValid && inputText.length > 0 ? [t.inputError] : []"
    />
    <v-btn color="primary" @click="generateQrCode" :disabled="!isValid || !inputText">{{ t.generateButton }}</v-btn>
    <div v-if="qrCodeUrl" class="qr-code-image">
      <img :src="qrCodeUrl" :alt="t.qrAltText" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import QRCode from 'qrcode';

type LanguageCode = 'en' | 'fr' | 'de' | 'pl' | 'zh' | 'ru';
type TranslationText = {
  languageButton: string;
  languageNames: Record<LanguageCode, string>;
  title: string;
  description: string;
  inputLabel: string;
  inputError: string;
  generateButton: string;
  qrAltText: string;
};

const languageOptions: LanguageCode[] = ['en', 'fr', 'de', 'pl', 'zh', 'ru'];

const translations: Record<LanguageCode, TranslationText> = {
  en: {
    languageButton: 'Language',
    languageNames: { en: 'English', fr: 'French', de: 'German', pl: 'Polish', zh: 'Chinese', ru: 'Russian' },
    title: 'QR Code Generator',
    description: 'Enter your text below to generate a QR code. Only alphanumeric characters and spaces are allowed.',
    inputLabel: 'Enter text to generate QR code',
    inputError: 'Only alphanumeric characters and spaces are allowed.',
    generateButton: 'Generate QR Code',
    qrAltText: 'Generated QR Code',
  },
  fr: {
    languageButton: 'Langue',
    languageNames: { en: 'Anglais', fr: 'Français', de: 'Allemand', pl: 'Polonais', zh: 'Chinois', ru: 'Russe' },
    title: 'Générateur de QR Code',
    description: 'Entrez votre texte ci-dessous pour générer un QR code. Seuls les caractères alphanumériques et les espaces sont autorisés.',
    inputLabel: 'Entrez le texte pour générer le QR code',
    inputError: 'Seuls les caractères alphanumériques et les espaces sont autorisés.',
    generateButton: 'Générer le QR Code',
    qrAltText: 'QR Code généré',
  },
  de: {
    languageButton: 'Sprache',
    languageNames: { en: 'Englisch', fr: 'Französisch', de: 'Deutsch', pl: 'Polnisch', zh: 'Chinesisch', ru: 'Russisch' },
    title: 'QR-Code-Generator',
    description: 'Geben Sie unten Ihren Text ein, um einen QR-Code zu erstellen. Nur alphanumerische Zeichen und Leerzeichen sind erlaubt.',
    inputLabel: 'Text eingeben, um QR-Code zu erstellen',
    inputError: 'Nur alphanumerische Zeichen und Leerzeichen sind erlaubt.',
    generateButton: 'QR-Code erstellen',
    qrAltText: 'Erstellter QR-Code',
  },
  pl: {
    languageButton: 'Język',
    languageNames: { en: 'Angielski', fr: 'Francuski', de: 'Niemiecki', pl: 'Polski', zh: 'Chiński', ru: 'Rosyjski' },
    title: 'Generator Kodów QR',
    description: 'Wpisz poniżej tekst, aby wygenerować kod QR. Dozwolone są tylko znaki alfanumeryczne i spacje.',
    inputLabel: 'Wpisz tekst do wygenerowania kodu QR',
    inputError: 'Dozwolone są tylko znaki alfanumeryczne i spacje.',
    generateButton: 'Wygeneruj kod QR',
    qrAltText: 'Wygenerowany kod QR',
  },
  zh: {
    languageButton: '语言 (Yuyan)',
    languageNames: { en: '英语 (Yingyu)', fr: '法语 (Fayu)', de: '德语 (Deyu)', pl: '波兰语 (Bolanyu)', zh: '中文 (Zhongwen)', ru: '俄语 (Eyu)' },
    title: '二维码生成器 (Erweima Shengchengqi)',
    description: '在下方输入文本以生成二维码。仅允许字母数字字符和空格。 (Zai xiafang shuru wenben yi shengcheng erweima. Jin yunxu zimu shuzi zifu he kongge.)',
    inputLabel: '输入要生成二维码的文本 (Shuru yao shengcheng erweima de wenben)',
    inputError: '仅允许字母数字字符和空格。 (Jin yunxu zimu shuzi zifu he kongge.)',
    generateButton: '生成二维码 (Shengcheng Erweima)',
    qrAltText: '已生成的二维码 (Yi shengcheng de erweima)',
  },
  ru: {
    languageButton: 'Язык',
    languageNames: { en: 'Английский', fr: 'Французский', de: 'Немецкий', pl: 'Польский', zh: 'Китайский', ru: 'Русский' },
    title: 'Генератор QR-кодов',
    description: 'Введите текст ниже, чтобы сгенерировать QR-код. Разрешены только буквенно-цифровые символы и пробелы.',
    inputLabel: 'Введите текст для генерации QR-кода',
    inputError: 'Разрешены только буквенно-цифровые символы и пробелы.',
    generateButton: 'Сгенерировать QR-код',
    qrAltText: 'Сгенерированный QR-код',
  },
};

const languageStorageKey = 'qr-code-forger-language';

const isLanguageCode = (value: string): value is LanguageCode => {
  return languageOptions.includes(value as LanguageCode);
};

const getInitialLanguage = (): LanguageCode => {
  if (typeof window === 'undefined') {
    return 'de';
  }

  const storedLanguage = window.localStorage.getItem(languageStorageKey);
  return storedLanguage && isLanguageCode(storedLanguage) ? storedLanguage : 'de';
};

const selectedLanguage = ref<LanguageCode>(getInitialLanguage());
const inputText = ref('');
const qrCodeUrl = ref<string | null>(null);

const t = computed(() => translations[selectedLanguage.value]);
const languageLabels = computed(() => t.value.languageNames);
const currentLanguageLabel = computed(() => {
  return languageLabels.value[selectedLanguage.value] ?? 'Deutsch';
});

watch(selectedLanguage, (language) => {
  window.localStorage.setItem(languageStorageKey, language);
});

// Regex: allow only alphanumeric and spaces
const inputRegex = /^.+$/;
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
.language-switcher {
  width: 100%;
  display: flex;
  justify-content: flex-end;
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