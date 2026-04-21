---
title: "Building Pahadi AI — A RAG-Powered Garhwali Chatbot for an Endangered Language"
date: 2026-04-21T10:00:00+05:30
draft: false
tags: ["AI", "RAG", "LLM", "Node.js", "System Design", "Garhwali", "Vector Search"]
categories: ["AI", "Backend"]
description: "How I built Pahadi AI — a Garhwali-speaking assistant on PahadiTube — using multi-tier caching, RAG over a hand-curated glossary, and a 3-provider LLM fallback chain. All running on free tiers."
summary: "Deep dive into the architecture of Pahadi AI: token-bucket rate limiting, semantic caching with Upstash Vector, RAG over 4,200+ Garhwali entries, and a Groq → OpenRouter → Gemini fallback chain — built to keep an endangered language alive in the digital age."
cover:
  image: ""
  alt: "Pahadi AI architecture"
  caption: ""
ShowToc: true
TocOpen: false
---

## The "Why" — A Language Worth Saving

UNESCO classifies **Garhwali, Kumaoni, and Jaunsari** as *critically endangered*. As a Garhwali speaker building [PahadiTube](https://garhwali-stream.onrender.com) — an OTT platform for Garhwali music, films, and folk culture — I wanted to add an AI assistant that doesn't just *understand* Garhwali, but **speaks it authentically**, in proper Devanagari, without bleeding Hindi grammar into every other sentence.

Off-the-shelf LLMs fail at this. Ask ChatGPT to write in Garhwali and you'll get Hindi with a few cosmetic substitutions ("है" → "च"). They also confidently hallucinate fake news, fake recipes, and fake folk traditions.

This post walks through how **Pahadi AI** solves both problems — using RAG, multi-tier caching, and a multi-provider fallback chain, all on free tiers.

---

## The Architecture in One Picture

```
User types
   ↓
[Per-IP rate limit]  →  [Message validation]
   ↓
[L1: Exact-hash cache?]      → HIT → stream cached SSE
   ↓ MISS
[L2: Semantic vector cache?] → HIT → stream cached SSE
   ↓ MISS
[RAG: glossary + grammar + few-shot + phrases + folk + lessons + news + memory]
   ↓
[Build mega system prompt]
   ↓
Groq (Llama 3.3 70B) → OpenRouter → Gemini 2.0 Flash → Memory fallback
   ↓
[Stream tokens to client via SSE]
   ↓
[On complete: cache + Redis log + vector upsert]
```

Stack:
- **Backend:** Node.js + Express
- **LLMs:** Groq (primary), OpenRouter (fallback), Google Gemini (last resort)
- **Vector store:** Upstash Vector (`mxbai-embed-large` / `bge-m3`)
- **Cache & memory:** Redis (Upstash) + in-process `node-cache`
- **Frontend:** React + SSE streaming (token-by-token UI)

---

## Problem 1 — Quota Protection

Groq's free tier is generous (~30 req/min, ~1000 req/day) but a single chatty bot can burn it in minutes. Two safeguards:

### Per-IP Token Bucket

```js
const IP_BURST = 8;
const IP_REFILL_MS = 6000; // 1 token / 6s ≈ 10 req/min/IP

function takeIpToken(ip) {
  const now = Date.now();
  let b = ipBuckets.get(ip) ?? { tokens: IP_BURST, last: now };
  const refill = Math.floor((now - b.last) / IP_REFILL_MS);
  if (refill > 0) {
    b.tokens = Math.min(IP_BURST, b.tokens + refill);
    b.last = now;
  }
  if (b.tokens <= 0) return false;
  b.tokens -= 1;
  ipBuckets.set(ip, b);
  return true;
}
```

### Global Circuit Breaker

When Groq returns 429, I parse its `"Please try again in 1h19m9.4s"` hint and trip the breaker for that exact duration. While tripped, all requests skip Groq entirely and try the next provider. No point hammering an API you know will fail.

```js
function parseRetryAfterMs(errText) {
  const m = /try again in\s+([0-9hms.\s]+?)(?:[."\s]|$)/i.exec(errText);
  if (!m) return null;
  let ms = 0;
  const h = /([0-9.]+)\s*h/i.exec(m[1]);   if (h) ms += +h[1] * 3600_000;
  const mm = /([0-9.]+)\s*m(?!s)/i.exec(m[1]); if (mm) ms += +mm[1] * 60_000;
  const s = /([0-9.]+)\s*s/i.exec(m[1]);   if (s) ms += +s[1] * 1000;
  return ms || null;
}
```

---

## Problem 2 — Three-Tier Caching

Every cache hit is one less token spent on a paid (or rate-limited) LLM.

### L1 — Exact Match (24h)

A SHA-256 hash of the last 5 normalized messages keys a `node-cache` entry. Same conversation tail → instant reply.

### L2 — Semantic Cache via Upstash Vector

Past Q→A pairs are embedded and stored. When a new question arrives, cosine similarity finds near-duplicates.

The catch: **embedding similarity alone is not enough**. Multilingual embedders score "गढ़वाली खाना बता" and "गढ़वाली नेगी जी बता" at ~0.90 — both are about *something* Garhwali, both have the same stylistic tail. Returning a cached food answer to a question about a singer would be a disaster.

So I gate on **two** signals:

```js
const SEMANTIC_CACHE_THRESHOLD = 0.88;  // cosine
const SEMANTIC_MIN_OVERLAP    = 0.5;   // Jaccard on content tokens

const semanticHit = memoryHits.find((h) =>
  h.score >= SEMANTIC_CACHE_THRESHOLD &&
  overlapRatio(query, h.q) >= SEMANTIC_MIN_OVERLAP  // stopwords removed
);
```

The token-overlap check filters the stylistic-words trap. Both gates must pass.

### L3 — Keyword Mirror

Redis-backed history is mirrored in-process so even when Vector is down, keyword overlap can serve a recent answer.

---

## Problem 3 — RAG to Stop Hallucinations and Hindi-bleed

This is where the real lift happens. Each request injects a budget-aware bundle of grounded context into the system prompt:

| Source | Size | What it grounds |
|---|---|---|
| **Curated glossary** | ~600 hand-written entries | Authentic Garhwali words with Hindi + English + tags |
| **Grammar reference** | Pronouns, copula, cases | Stops "है/हैं" creeping in |
| **Himlingo dictionary** | ~4,200 EN→GW pairs (scraped) | Coverage for obscure English words |
| **Few-shot pairs** | Hand-curated + generated | Hindi → Garhwali style transfer examples |
| **Phrase book** | EN → Roman Garhwali | Conversational patterns |
| **Folk stories** | Tilu Rauteli, Jagdev Panwar… | Real cultural narratives, not invented ones |
| **Lessons** | 10-day curriculum | Greetings, numbers, tense |
| **Live news** | Redis CMS articles | "आजकल क्या हो रौ" answered with real headlines |
| **Past conversations** | Vector + Redis | Long-term memory |

### Romanization-aware Retrieval

People type Garhwali in Roman script with random spellings — `duur`, `dur`, `duuur`, all mean दूर. The trick is folding repeated Latin letters down to one occurrence in **both** the index and the query:

```js
function collapseRoman(text) {
  return String(text || '').toLowerCase().replace(/([a-z])\1+/g, '$1');
}
// "duuur" / "duur" / "dur" all collapse to "dur" → same glossary row
```

Devanagari is untouched, so this is a strict recall boost — never loses verbatim matches.

### News RAG to Kill Hallucinations

Detect intent, then hard-anchor the model:

```js
const NEWS_INTENT_RE = /(आजकल|aaj\s*kal|aajkal|today|latest|ताज़ा|समाचार|news|खबर|happening)/i;
```

If matched, the top 5 articles are injected with a strict prompt instruction:

> "अगर यि खबरें उपयोगकर्ता कि बात सी मेल नी खांदी, त ईमानदारी सी बोल 'मीं तैं ताज़ा खबर कु पता नी च' — कभी कोई नी बणा।"

(*"If these articles don't match the user's question, honestly say 'I don't have current news' — never make any up."*)

---

## Problem 4 — A System Prompt That Actually Enforces Style

A 12 KB system prompt sounds excessive until you realize how stubbornly LLMs default to Hindi. The prompt covers:

- **Intent detection** — "How do you say X in Garhwali?" → reply in *only* Garhwali, no English gloss
- **Romanized Garhwali detection** — `kakh ja rou che` → treat as `कख जा रौ छ`
- **Output language matrix:**
  - Garhwali query → Garhwali only
  - Hindi query → Hindi + Garhwali both
  - English query → English + Garhwali both
- **Hindi-replacement table** with ~30 rows: `है → छ`, `करते हैं → करदा छां`, `के साथ → सँग`…
- **Word-invention guard** — if you don't know an authentic Garhwali word, **use Hindi and admit it**, don't invent
- **Cultural facts list** — Narendra Singh Negi (singer), recipes (kapa is *vegetarian*), festivals (Harela, Phooldei)
- **Reference answers** — full worked examples of *correct* and *incorrect* style, side by side

That last one is the killer feature. Showing the model exactly what to avoid (with annotated mistakes) outperforms 10 abstract rules.

---

## Problem 5 — Multi-Provider Fallback

Free tiers fail. Make peace with it.

```js
// Pseudocode
if (allProvidersCoolingDown) return memoryFallback();

let ok = await tryStreamProvider(GROQ, payload, res, …);
if (!ok && !openrouterCoolingDown) ok = await tryStreamProvider(OPENROUTER, …);
if (!ok && !geminiCoolingDown)     ok = await tryStreamGemini(payload, res, …);
if (!ok)                           streamFallbackReply(res, await findFallbackReply(query));
```

All three speak streaming — Groq and OpenRouter use OpenAI's wire format (one helper handles both); Gemini has its own `streamGenerateContent` shape with `contents` + `systemInstruction`, so it gets a separate adapter that translates back to the same SSE `data: {delta}` envelope the frontend expects.

The client never knows which provider answered. An `X-Provider` response header makes debugging obvious.

---

## Problem 6 — Honest Streaming with Truncation Detection

When `finish_reason === 'length'` (max_tokens hit), the reply is **not cached**. Otherwise the user gets the same cut-off mid-sentence response every time they ask. A simple guard:

```js
if (finish && finish !== 'stop') truncated = true;
// later …
if (!clientClosed && fullReply.length > 5 && !truncated) {
  setCached(safeMessages, fullReply);
  persistExchange(lastUser?.content, fullReply);
}
```

---

## Lessons Learned

1. **RAG is more about *what* you retrieve than *how*.** A hand-curated 600-entry glossary outperforms a generic 50K-entry dump because the curated entries carry tags, Hindi, English, and notes — the model can latch onto authoritative grounding.
2. **Embedding similarity alone is dangerous for short queries.** Always pair it with a token-overlap check.
3. **Prompt with examples, not just rules.** A side-by-side "wrong vs. right" answer beats any number of "do this, not that" bullets.
4. **Free tiers + circuit breakers + fallbacks = production-grade reliability** without spending a rupee.
5. **Cultural correctness is part of correctness.** Hallucinating that *kapa* contains meat (it's strictly vegetarian) isn't a "minor error" — it's a credibility-killer. RAG fixes it.

---

## What's Next

- **Voice** — Garhwali TTS is the next frontier. Web Speech API works for input but lacks Garhwali voices for output.
- **Crowdsourced glossary** — Let native speakers contribute entries with one-tap moderation.
- **Kumaoni + Jaunsari** — Same architecture, sister languages.

If you'd like to chat with Pahadi AI yourself, it's live at [PahadiTube → Pahadi AI](https://garhwali-stream.onrender.com/pahadi-ai). Feedback (especially from Garhwali speakers!) is gold.

---

*हमारी भाषा हमारी पहचान च। यो AI एक छोटु प्रयास च — यि भाषा तैं डिजिटल युग मा जिंदा रखणु। तुम भी अपणा घर मा गढ़वळि बोल्या। 🙏*
