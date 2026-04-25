# 🧠 Analogical Learning System
### DSU GenAI Research Project

> An AI-powered educational platform that explains complex STEM concepts through personalized analogies — running **entirely in your browser** with no API key, no server, and no cost.

---

## 📌 Overview

The **Analogical Learning System** is a single-file web application built for the DSU (Dayananda Sagar University) GenAI research project. It takes a student's hobbies and academic interests, then uses a locally-running language model to generate personalized analogies that make difficult science concepts easier to understand.

**Example:** A student who loves Cricket asking about Newton's Laws will get an explanation framed around bowling, batting, and fielding — not abstract textbook definitions.

---

## ✨ Features

| Feature | Description |
|---|---|
| 🤖 On-Device AI | Runs `LaMini-Flan-T5-248M` locally via WebAssembly — zero API calls |
| 🎯 Personalized Analogies | Explanations tailored to the student's hobbies (Cricket, Gaming, Music, etc.) |
| 📚 NLP Pipeline | Tokenization, context analysis, and concept mapping all run in-browser |
| 🎬 Video Script Generator | Produces production-ready scripts for educational YouTube/Reels content |
| 📶 Offline Support | After the first load, the model is cached and works without internet |
| 🔒 Privacy-First | No data ever leaves the browser — all inference is local |

---

## 🏗️ Architecture

```
index.html
│
├── ES Module (type="module")
│   └── @xenova/transformers  ──► Loads ONNX model from HuggingFace CDN
│       └── Xenova/LaMini-Flan-T5-248M
│           ├── Downloaded once (~500 MB)
│           └── Cached via browser Cache API (works offline after)
│
├── React 18 (UMD via CDN)
│   ├── Welcome  →  Profile  →  Query  →  Processing  →  Result
│   └── Stages managed via useState
│
└── NLP Helpers (pure JS, no libraries)
    ├── preprocessInput()       — tokenization & normalization
    ├── analyzeUserContext()    — hobby/interest extraction
    ├── mapConceptToAnalogy()   — subject classification
    └── generateExplanation()  — calls local model pipeline
```

---

## 🤖 The Local Model

### Model: `Xenova/LaMini-Flan-T5-248M`

| Property | Value |
|---|---|
| Architecture | T5 (seq2seq / text2text) |
| Parameters | ~248 Million |
| Training | Instruction-tuned on LaMini dataset |
| Runtime | ONNX via WebAssembly (`@xenova/transformers`) |
| Size on disk | ~500 MB (cached in browser after first download) |
| HuggingFace Hub | [Xenova/LaMini-Flan-T5-248M](https://huggingface.co/Xenova/LaMini-Flan-T5-248M) |

### Why this model?
- **Instruction-following:** LaMini-Flan-T5 is fine-tuned on 2.58M instruction-response pairs, making it excellent at following natural language prompts like *"Explain X using an analogy related to Y"*
- **Browser-compatible:** Small enough to run in WebAssembly without crashing the tab
- **No API key needed:** Unlike GPT-4 or Claude, this runs 100% client-side

### How inference works
```js
// 1. Import transformers (ES module, runs once)
import { pipeline } from '@xenova/transformers';

// 2. Load the pipeline (downloads + caches weights on first call)
const generator = await pipeline('text2text-generation', 'Xenova/LaMini-Flan-T5-248M');

// 3. Run inference locally
const output = await generator(prompt, { max_new_tokens: 300, do_sample: false });
```

---

## 🚀 Getting Started

### Prerequisites
- A modern browser (Chrome 90+, Firefox 90+, Edge 90+, Safari 15.4+)
- ~500 MB of free storage for the model cache (first run only)
- Internet connection for the first load (to download model weights)

### Running the app

Since this is a single HTML file, just open it:

```bash
# Option 1 — Open directly
open index.html

# Option 2 — Serve locally (recommended, avoids CORS issues with some browsers)
python -m http.server 8080
# then visit http://localhost:8080

# Option 3 — Using Node
npx serve .
# then visit http://localhost:3000
```

> **Note:** Serving via a local HTTP server (`python -m http.server` or `npx serve`) is recommended over opening the file directly, as some browsers restrict ES module imports from `file://` URLs.

---

## 🗺️ User Flow

```
1. Welcome Screen
   └── Overview of features + model info banner

2. Profile Setup
   ├── Enter your name
   ├── Select hobbies (Cricket, Gaming, Music, etc.)
   └── Select academic interests (Physics, Biology, etc.)

3. Query Input
   ├── Model begins downloading in background (progress bar shown)
   ├── Type a scientific concept or pick a quick example
   └── Click "Generate Explanation"

4. Processing Pipeline (animated steps)
   ├── Step 1: Preprocessing — tokenize & normalize query
   ├── Step 2: Context Analysis — extract primary hobby & interests
   ├── Step 3: Analogy Mapping — classify subject (physics/bio/chem)
   ├── Step 4: Local Model Inference — run LaMini-Flan-T5 on-device
   └── Step 5: Finalizing — format output

5. Result
   ├── Personalized explanation with hobby-based analogy
   ├── Video Script Generator (60–90 second production guide)
   └── Copy to clipboard
```

---

## 📁 Project Structure

```
analogical-learning-system/
└── index.html          # Entire application — self-contained single file
└── README.md           # This file
```

The project is intentionally a single file to make it easy to share, submit, and run without a build step.

---

## 🧩 Tech Stack

| Layer | Technology |
|---|---|
| UI Framework | React 18 (UMD, loaded via CDN) |
| Styling | Tailwind CSS + custom CSS |
| Icons | Font Awesome 6 |
| Fonts | Inter (Google Fonts) |
| Transpiler | Babel Standalone (in-browser JSX) |
| ML Runtime | `@xenova/transformers` v2.17.2 |
| Model Format | ONNX (Open Neural Network Exchange) |
| Execution | WebAssembly (WASM) |

---

## 🧪 Supported Concepts (Examples)

The model handles any free-text query, and the NLP classifier provides extra routing for common STEM topics:

| Query | Subject Detected | Example Hobby Used |
|---|---|---|
| "Explain Newton's Laws" | Physics | Cricket — bowling & batting |
| "How does photosynthesis work?" | Biology | General solar factory metaphor |
| "What are atoms made of?" | Chemistry | Solar system analogy |
| "Explain chemical reactions" | Chemistry | Cooking analogy |
| Any other STEM topic | General | Adapts to selected hobby |

---

## ⚡ Performance Notes

- **First load:** Model download takes 1–5 minutes depending on connection speed (~500 MB)
- **Subsequent loads:** Instant — weights are served from the browser's Cache API
- **Inference time:** ~5–30 seconds per query (depends on device CPU/GPU)
- **RAM usage:** ~600–800 MB during inference
- **Offline:** Fully functional after initial download

### Tips for faster inference
- Use Chrome or Edge — they have the most optimized WebAssembly runtimes
- Close other heavy browser tabs to free up RAM
- On mobile, inference may be slow or fail on low-end devices (≤4 GB RAM)

---

## 🔄 Fallback Behaviour

If the local model fails to load (unsupported browser, insufficient memory, etc.), the app automatically falls back to a **deterministic template engine** that generates structured explanations for common topics (Newton's Laws, Photosynthesis, Atoms). No error is shown to the user — the fallback is seamless.

---

## 🛠️ Customization

### Swap the model
Replace the model ID in the `getTextGenPipeline` call to use a different Xenova-compatible model:

```js
// In the <script type="module"> block:
'Xenova/LaMini-Flan-T5-248M'   // current — balanced quality/speed

// Alternatives:
'Xenova/flan-t5-small'          // faster, lower quality (~80 MB)
'Xenova/flan-t5-base'           // similar size, less instruction-tuned
'Xenova/LaMini-Flan-T5-77M'    // smallest, fastest (~150 MB)
```

### Add more hobbies or subjects
Edit the arrays inside the `App` component:
```js
const hobbies   = ['Cricket', 'Football', 'Gaming', ...];
const interests = ['Physics', 'Chemistry', 'Biology', ...];
```

### Adjust generation parameters
```js
const output = await generator(prompt, {
    max_new_tokens: 300,   // increase for longer explanations
    do_sample: false,      // set true + add temperature for more creative output
    temperature: 0.7,      // only used when do_sample: true
});
```

---

## 📄 License

This project was created as part of the DSU GenAI coursework. Free to use and modify for educational purposes.

---

## 🙏 Acknowledgements

- [Xenova/transformers.js](https://github.com/xenova/transformers.js) — for making browser-based ML inference possible
- [MBZUAI/LaMini-Flan-T5-248M](https://huggingface.co/MBZUAI/LaMini-Flan-T5-248M) — the original model authors
- [HuggingFace](https://huggingface.co) — for hosting the ONNX-converted model weights
- DSU Faculty — for the GenAI project framework

---

*Built with ❤️ for DSU GenAI Research Project — running AI, locally, for everyone.*
