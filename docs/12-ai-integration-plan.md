# DrawSaver — AI Future Integration Plan

> **Version:** 1.0  
> **Last Updated:** 2026-05-27  
> **Status:** Planning (Phase 5+)  

---

## 1. AI Feature Roadmap

| Feature | Priority | Complexity | Dependencies | Target Phase |
|---------|----------|------------|-------------|-------------|
| Smart Shape Correction | P2 | Medium | TensorFlow.js | Phase 5a |
| AI Color Palette Suggestions | P2 | Low | OpenAI API | Phase 5a |
| Sketch-to-Clean-Art | P3 | High | Stable Diffusion API | Phase 5b |
| AI Drawing Assist (autocomplete) | P3 | High | Custom ML model | Phase 5c |
| Handwriting-to-Text (OCR) | P3 | Medium | Tesseract.js / Google Vision | Phase 5b |
| AI Whiteboard Summarizer | P3 | Medium | OpenAI GPT API | Phase 5b |
| Background Removal | P2 | Low | remove.bg API / U2-Net | Phase 5a |

---

## 2. Smart Shape Correction

### What It Does
When a user freehand-draws a shape (rough circle, rectangle, triangle, arrow), the system detects the intended shape and replaces the freehand path with a clean geometric shape.

### Implementation Strategy

**Approach: Client-Side ML with TensorFlow.js**

```
User draws rough shape → Capture path points → Run through classifier
→ If confidence > 80% → Replace with clean Fabric.js shape
→ If confidence < 80% → Keep original freehand path
```

**Steps:**
1. Collect path points from Fabric.js `path:created` event
2. Extract features: bounding box ratio, point count, curvature, closure
3. Classify using a lightweight CNN trained on Google QuickDraw dataset
4. Map classification to Fabric.js shape constructor

```javascript
// Pseudocode for shape correction
canvas.on('path:created', async (e) => {
  const path = e.path;
  const points = path.path; // Array of [command, x, y, ...] segments
  
  if (shapeCorrection.enabled) {
    const prediction = await shapeClassifier.predict(points);
    
    if (prediction.confidence > 0.8) {
      const bounds = path.getBoundingRect();
      let cleanShape;
      
      switch (prediction.label) {
        case 'circle':
          cleanShape = new fabric.Circle({
            radius: Math.max(bounds.width, bounds.height) / 2,
            left: bounds.left, top: bounds.top,
            stroke: path.stroke, fill: 'transparent'
          });
          break;
        case 'rectangle':
          cleanShape = new fabric.Rect({
            width: bounds.width, height: bounds.height,
            left: bounds.left, top: bounds.top,
            stroke: path.stroke, fill: 'transparent'
          });
          break;
        case 'triangle':
          cleanShape = new fabric.Triangle({ /* ... */ });
          break;
        case 'arrow':
          cleanShape = createArrowShape(points);
          break;
      }
      
      if (cleanShape) {
        canvas.remove(path);
        canvas.add(cleanShape);
        // Show brief "Shape corrected ✨" toast
      }
    }
  }
});
```

**Training Data:** Google QuickDraw dataset (50M+ drawings across 345 categories). Use a subset: circle, rectangle, triangle, line, arrow, star.

**Model Size:** ~2MB (quantized TensorFlow.js model), loaded on demand when shape correction is enabled.

---

## 3. AI Color Palette Suggestions

### What It Does
Analyzes the current canvas colors and suggests harmonious palettes, or generates palettes from a text prompt.

### Implementation

**Approach: OpenAI API for text-to-palette + algorithmic harmony**

```javascript
// API endpoint: POST /api/v1/ai/palette
const generatePalette = async (req, res) => {
  const { prompt, existingColors, mode } = req.body;
  
  if (mode === 'from-prompt') {
    // Use OpenAI to generate palette from text description
    const response = await openai.chat.completions.create({
      model: 'gpt-4o-mini',
      messages: [{
        role: 'system',
        content: 'Generate a color palette of exactly 5 hex colors based on the user description. Return ONLY a JSON array of hex strings.'
      }, {
        role: 'user',
        content: `Generate a color palette for: "${prompt}"`
      }],
      response_format: { type: 'json_object' },
      max_tokens: 100
    });
    
    const palette = JSON.parse(response.choices[0].message.content);
    return res.json({ success: true, data: { colors: palette.colors } });
  }
  
  if (mode === 'harmonious') {
    // Algorithmic: generate complementary/analogous/triadic from existing colors
    const palette = generateHarmoniousPalette(existingColors);
    return res.json({ success: true, data: { colors: palette } });
  }
};
```

**UI Integration:**
- "Suggest Palette" button in color panel
- Text input: "sunset over mountains" → generates warm palette
- Auto-detect: analyze canvas dominant colors → suggest complementary

### Cost Estimate
- GPT-4o-mini: ~$0.15 per 1M input tokens
- ~50 tokens per palette request = ~$0.00001 per request
- 10,000 requests/month = ~$0.10/month (negligible)

---

## 4. Sketch-to-Clean-Art

### What It Does
User draws a rough sketch, AI generates a polished version while preserving the composition and intent.

### Implementation Strategy

**Approach: Stable Diffusion API (img2img endpoint)**

```
┌─────────────┐      ┌──────────────┐      ┌──────────────┐
│ User Sketch │─────►│ Export as    │─────►│ Stable       │
│ (Canvas)    │      │ PNG (base64) │      │ Diffusion    │
│             │      │              │      │ img2img API  │
└─────────────┘      └──────────────┘      └──────┬───────┘
                                                   │
                     ┌──────────────┐      ┌───────▼──────┐
                     │ Import to    │◄─────│ Generated    │
                     │ Canvas as    │      │ Image        │
                     │ new layer    │      │ (base64)     │
                     └──────────────┘      └──────────────┘
```

**Backend Endpoint:**

```javascript
// POST /api/v1/ai/sketch-to-art
const sketchToArt = async (req, res) => {
  const { imageBase64, prompt, style, strength } = req.body;
  
  // Validate image size (max 2MB)
  if (Buffer.byteLength(imageBase64, 'base64') > 2 * 1024 * 1024) {
    return res.status(400).json({ error: 'Image too large' });
  }
  
  const response = await fetch('https://api.stability.ai/v1/generation/stable-diffusion-xl-1024-v1-0/image-to-image', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.STABILITY_API_KEY}`,
      'Content-Type': 'multipart/form-data'
    },
    body: formData({
      init_image: imageBase64,
      text_prompts: [{ text: prompt || 'clean professional illustration', weight: 1 }],
      image_strength: strength || 0.35, // 0-1, how much to change
      style_preset: style || 'digital-art',
      cfg_scale: 7,
      samples: 1,
      steps: 30
    })
  });
  
  const result = await response.json();
  const generatedImage = result.artifacts[0].base64;
  
  // Store result in S3
  const imageUrl = await uploadToS3(generatedImage, `ai-art/${req.user.sub}/${Date.now()}.png`);
  
  res.json({ success: true, data: { imageUrl, base64: generatedImage } });
};
```

**Style Presets Available:**
- `digital-art` — Clean digital illustration
- `anime` — Anime/manga style
- `photographic` — Realistic rendering
- `comic-book` — Comic style
- `line-art` — Clean linework

### Cost & Rate Limiting
- Stability AI: ~$0.002–0.006 per generation
- Rate limit: 5 generations per user per hour (free tier), 50/hour (premium)
- Async processing: queue via BullMQ, notify via WebSocket when complete

---

## 5. AI Drawing Assist (Autocomplete)

### What It Does
As the user draws, AI predicts and suggests the next strokes, similar to autocomplete in code editors. User can accept (Tab) or dismiss (Esc).

### Implementation Approach

**Phase 1: Template Matching**
- Detect partial shapes (half-circle, L-shape) and suggest completion
- Rules-based, no ML required
- E.g., detecting 3 sides of a rectangle → suggest 4th side

**Phase 2: ML-Based Prediction**
- Train on Google QuickDraw dataset sequences
- Use an RNN/LSTM model that predicts next stroke given previous strokes
- Run client-side via TensorFlow.js for instant predictions

```javascript
// Pseudocode for drawing autocomplete
let strokeBuffer = [];
let suggestionTimeout = null;

canvas.on('path:created', (e) => {
  strokeBuffer.push(extractStrokeFeatures(e.path));
  
  // Debounce: wait 500ms after last stroke to suggest
  clearTimeout(suggestionTimeout);
  suggestionTimeout = setTimeout(async () => {
    const suggestion = await drawingPredictor.predict(strokeBuffer);
    
    if (suggestion.confidence > 0.7) {
      // Render suggestion as semi-transparent ghost shape
      renderSuggestion(suggestion.path, suggestion.label);
    }
  }, 500);
});

// User presses Tab → accept suggestion
document.addEventListener('keydown', (e) => {
  if (e.key === 'Tab' && activeSuggestion) {
    e.preventDefault();
    commitSuggestion(activeSuggestion);
  }
});
```

---

## 6. Handwriting-to-Text (OCR)

### What It Does
Select handwritten text on the canvas → convert to typed text using OCR.

### Implementation

**Option A: Client-Side (Tesseract.js)**
```javascript
import Tesseract from 'tesseract.js';

const recognizeText = async (imageData) => {
  const { data: { text, confidence } } = await Tesseract.recognize(
    imageData,
    'eng',
    { logger: m => console.log(m) }
  );
  return { text: text.trim(), confidence };
};
```
- Free, no API cost
- Slower (~2-5 seconds)
- Lower accuracy for messy handwriting

**Option B: Google Cloud Vision API**
- Higher accuracy
- ~$1.50 per 1,000 requests
- Better for production

---

## 7. AI Whiteboard Summarizer

### What It Does
Analyze the canvas content (text, shapes, layout) and generate a text summary of the whiteboard — useful for meeting notes.

### Implementation

```javascript
// 1. Export canvas as image
// 2. Send to GPT-4 Vision for analysis
const summarizeWhiteboard = async (canvasImageBase64) => {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [{
      role: 'user',
      content: [
        { 
          type: 'text', 
          text: 'Analyze this whiteboard image. Extract all text, describe diagrams, and provide a structured summary of the content. Format as bullet points.' 
        },
        { 
          type: 'image_url', 
          image_url: { url: `data:image/png;base64,${canvasImageBase64}` } 
        }
      ]
    }],
    max_tokens: 1000
  });
  
  return response.choices[0].message.content;
};
```

---

## 8. Architecture for AI Features

```
┌────────────────────────────────────────────────────────────┐
│                    AI Service Layer                         │
│                                                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │ Shape        │  │ Palette      │  │ Sketch-to    │    │
│  │ Correction   │  │ Suggestions  │  │ Art          │    │
│  │ (Client-side)│  │ (Server API) │  │ (Server API) │    │
│  │ TensorFlow.js│  │ OpenAI       │  │ Stability AI │    │
│  └──────────────┘  └──────────────┘  └──────┬───────┘    │
│                                              │            │
│                                     ┌────────▼────────┐   │
│                                     │   BullMQ Queue  │   │
│                                     │  (async process)│   │
│                                     └────────┬────────┘   │
│                                              │            │
│                                     ┌────────▼────────┐   │
│                                     │   S3 Storage    │   │
│                                     │  (AI outputs)   │   │
│                                     └─────────────────┘   │
└────────────────────────────────────────────────────────────┘
```

### Key Principles
1. **AI features are always optional** — never block core drawing functionality
2. **Client-side when possible** — shape correction, OCR run in browser (no latency, no cost)
3. **Async for heavy processing** — sketch-to-art uses job queue, notifies via WebSocket
4. **Cost controls** — rate limit AI API calls per user, offer premium tiers
5. **Graceful degradation** — if AI service is down, drawing continues normally

### Cost Management

| Feature | API | Cost per Use | Free Tier Limit | Premium Limit |
|---------|-----|-------------|-----------------|---------------|
| Shape Correction | None (client) | $0 | Unlimited | Unlimited |
| Color Palette | OpenAI | ~$0.0001 | 20/day | 200/day |
| Sketch-to-Art | Stability AI | ~$0.004 | 5/day | 50/day |
| Whiteboard Summary | OpenAI GPT-4o | ~$0.01 | 3/day | 30/day |
| Background Removal | remove.bg | ~$0.05 | 3/day | 50/day |
