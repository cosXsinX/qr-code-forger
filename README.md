# QR Code Forger

A simple web application that allows you to generate QR codes from your own text input. Enter your desired text, validate it, and instantly get a QR code image you can use anywhere.

## Prerequisites
- **Node.js** (v16 or higher recommended)
- **npm** or **yarn** (for installing dependencies and running scripts)

## Features
- Generate QR codes from custom text
- Input validation (alphanumeric and spaces)
- Modern UI with Vuetify

## Getting Started

### Install
```bash
npm install
# or
yarn install
```

### Run (Development Server)
```bash
npm run dev
# or
yarn dev
```
This will start the app locally, usually at [http://localhost:5173](http://localhost:5173) or the port shown in your terminal.

### Build for Production
```bash
npm run build
# or
yarn build
```

### Lint
```bash
npm run lint
# or
yarn lint
```

## Usage
1. Enter your text in the input field.
2. Click "Generate QR Code" to see the QR code image.

## Project Structure
- `src/components/QrCodeGenerator.vue`: Main QR code generator component
- `src/pages/index.vue`: Main page integrating the generator

## License
MIT
