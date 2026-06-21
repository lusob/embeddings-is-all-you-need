# Embeddings is all you need

Local semantic intent classification in your browser. No API calls, no server, no fine-tuning.

**Live demo:** https://lusob.github.io/embeddings-is-all-you-need/

## The idea

Turns out you dont need an LLM to understand what a user wants. If you have good word embeddings, a simple cosine similarity check against a small dictionary of reference phrases is enough.

Say "turn off the lights" → browser computes its embedding → compares against embeddings of "lights off", "dark mode", "switch off" → picks the action. Done in < 50ms.

## How it works

### 1. Embeddings (the hard part that we outsourced)

A transformer model (`all-MiniLM-L6-v2`, 23MB WASM) runs locally to convert text to 384-dimensional vectors. These embeddings capture semantic meaning — words that mean similar things are close together in embedding space.

```
"turn off the lights"  →  [0.12, -0.34, 0.89, ...]
"lights off"           →  [0.11, -0.35, 0.88, ...]  (very close, same meaning)
"dark mode"            →  [0.13, -0.36, 0.87, ...]  (very close, same meaning)
"throw a ball"         →  [0.52, 0.22, -0.11, ...]  (far away, different meaning)
```

The model is downloaded once and cached in browser RAM. After that, inference is instant.

### 2. Intent matching (the trivial part)

For each action (intent), we store 2-3 reference phrases in English (or Spanish). When the user speaks or types:

1. Compute embedding of their input
2. Compare against all reference embeddings using cosine similarity
3. Pick the highest score above a confidence threshold (default 0.35)

That's it. No training, no API calls, no LLM.

### 3. Actions

The demo implements 13 intents:

- **GREET** — "hello" → waving hand animation
- **FIREWORKS** — "launch fireworks" → celebratory animation
- **MATRIX** — "activate matrix" → green rain effect
- **LAUNCH_OBJECT** — "throw a rocket" → bouncing emoji (matches keyword)
- **CHANGE_BACKGROUND** — "purple background" → solid color (or any CSS color)
- **LIGHT_MODE** / **DARK_MODE** — toggle dark/light theme
- **SEARCH** — "search for pasta" → open Google
- **ADD_ITEM** / **REMOVE_ITEM** → shopping list
- **CLEAR_LIST** — empty the list
- **TIMER** — "set a timer for 5 minutes" → countdown

You can also define **custom actions** with a webhook URL (good for IFTTT, Zapier, etc).

## Input methods

- **Voice** (Web Speech API) — "click the microphone" or Cmd+Shift+K in most Chrome/Edge browsers
- **Text** — type in the textarea and press Enter
- **Click examples** — pre-made phrases at the bottom
- **If speech isn't available** — error banner at top suggests using text input instead

## Why embeddings are enough

Classic intent classification needs:

1. **Training data** — hundreds of labeled examples per intent
2. **Fine-tuning** — hours of GPU time per model
3. **Deployment** — API server, authentication, rate limits
4. **Cost** — per-query billing

This approach needs:

1. **Reference phrases** — 2-3 examples per intent, written once
2. **Embedding model** — pre-trained on billions of words (already smart)
3. **Deployment** — one HTML file, zero servers
4. **Cost** — $0, forever

The trade-off: it only works for intent classification (not code generation, summarization, etc). But for "what does the user want me to do?", embedding similarity is shockingly powerful.

## Local-first philosophy

Everything runs in your browser:

- ✅ No data sent to any server
- ✅ Works offline (after first load)
- ✅ No accounts, no auth, no tracking
- ✅ Fast (no network round-trips)
- ✅ Private (your commands stay yours)

## Open questions

- **Ambiguity**: What if the user says something between two intents? Confidence threshold helps, but sometimes you need clarification. Returning top-3 candidates with scores could help UX.
- **Multi-step actions**: Some commands need context ("add milk", then next "add eggs" → stay in shopping mode). Current design treats each input independently.
- **Intent versioning**: Reference phrases drift over time. How to update them without re-deploying?

## Technical stack

- **Frontend**: Vanilla JavaScript, Tailwind CSS
- **Embeddings**: [@xenova/transformers](https://huggingface.co/docs/transformers.js) (Transformers.js)
- **Model**: [all-MiniLM-L6-v2](https://huggingface.co/Xenova/all-MiniLM-L6-v2) (384-dim, 23MB WASM)
- **Hosting**: GitHub Pages (static HTML)

## Similar projects

- [Neural Computers](https://arxiv.org/abs/2604.06425) — Meta AI's video model that predicts GUI state from pixels + user input
- [Browser ML](https://github.com/topics/browser-ml) — growing ecosystem of in-browser ML
- Classic NLU — RASA, Snips (now Sonos), Microsoft Luis

## Try it

```bash
git clone https://github.com/lusob/embeddings-is-all-you-need
cd embeddings-is-all-you-need
python3 -m http.server 8000
# Open http://localhost:8000
```

Or just visit the [live demo](https://lusob.github.io/embeddings-is-all-you-need/).
