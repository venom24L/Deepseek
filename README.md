# 🌑 Pixel Whisper

> **Hide secrets in plain sight.** A pure, offline-capable, zero-dependency web application for Image Steganography using the Least Significant Bit (LSB) technique.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Dependencies](https://img.shields.io/badge/dependencies-none-success.svg)
![Status](https://img.shields.io/badge/status-stable-brightgreen.svg)

## 📖 Overview

**Pixel Whisper** is a cyber-noir themed, single-page application that allows users to embed secret text messages inside PNG images and extract them later. It operates entirely in the browser using the HTML5 Canvas API and pure Vanilla JavaScript, ensuring 100% privacy and offline capability. No data ever leaves your machine.

## ✨ Features

- **Pure Vanilla JS:** Zero external libraries. The LSB algorithm is implemented from scratch.
- **Full UTF-16 Support:** Encode and decode any character, including emojis, Arabic, and CJK.
- **Self-Describing Binary Format:** Uses a 32-bit length header and 16-bit null delimiter for robust data extraction.
- **Real-time Capacity Tracking:** Visual progress bar showing exact character limits based on image dimensions.
- **Cyber-Noir UI:** Dark mode aesthetic with neon accents, drag-and-drop support, and smooth animations.
- **100% Offline & Private:** No server, no tracking, no telemetry. Everything runs locally in your browser.

## 🧠 How It Works: The Math & Bitwise Logic

The core engine manipulates the raw RGBA pixel data obtained via `ctx.getImageData()`. We only modify the **Red, Green, and Blue** channels, leaving the Alpha channel untouched to preserve image opacity.

### 1. The Binary Format
The message is serialized into a continuous bitstream:
```text
[ 32-bit Length Header ] + [ N × 16-bit UTF-16 Characters ] + [ 16-bit Null Delimiter ]
```
- **Header:** Tells the decoder exactly how many characters to read (supports up to ~4.2 billion chars).
- **Payload:** 16 bits per character for full Unicode support.
- **Delimiter:** A 16-bit zero sequence (`0x0000`) acting as a safety checksum.

### 2. Encoding (Bitwise Injection)
For every bit in our serialized stream, we take the next RGB channel value and inject the bit into its Least Significant Bit (LSB).

```javascript
// 1. Clear the LSB using a bitmask (0xFE = 11111110)
let clearedChannel = pixelValue & 0xFE; 

// 2. Set the LSB to our data bit (0 or 1) using bitwise OR
pixelValue = clearedChannel | dataBit;
```
*Visual Impact:* A pixel channel changing from `173` to `172` is a `0.4%` change in luminance. It is mathematically imperceptible to the human eye.

### 3. Decoding (Bitwise Extraction)
To read the data back, we isolate the LSB of each channel using a bitmask of `0x01` (00000001).

```javascript
// Extract the LSB
let extractedBit = pixelValue & 0x01;

// Reconstruct the number using left-shift accumulation
let charCode = (accumulator << 1) | extractedBit;
```

## 🚀 Usage

Since this is a pure front-end application, no build step is required.

1. Clone the repository:
```bash
   git clone https://github.com/your-username/your-repo-name.git
```
2. Open `index.html` directly in any modern web browser (Chrome, Firefox, Edge, Safari).
3. **To Hide:** Upload a PNG, type your secret, and click "Encode & Download".
4. **To Reveal:** Upload the encoded PNG and click "Decode Message".

*Note: Always use **PNG** or **BMP** formats. Lossy formats like JPEG will destroy the LSBs during compression, corrupting the hidden data.*

## ⚙️ Technical Constraints & Performance

- **Capacity Formula:** `Max Chars = floor((Width × Height × 3) - 48) / 16`
- **Non-Blocking UI:** Heavy pixel manipulation is deferred using `setTimeout` to allow the browser's event loop to render progress indicators before the main thread is blocked by the `for` loop.
- **Memory Management:** Uses `Uint8ClampedArray` via `ImageData` for direct, typed-array memory access, ensuring high performance even on large images (e.g., 4K resolution).

## 🛠️ Tech Stack

- **HTML5 Canvas API** (Pixel manipulation)
- **Vanilla JavaScript (ES6+)** (Logic & DOM)
- **CSS3** (Grid, Flexbox, Custom Properties, Animations)

## 📜 License

This project is open-source and available under the [MIT License](LICENSE). Feel free to use, modify, and distribute it as you see fit.

---

*Built with precision, designed for stealth.*
