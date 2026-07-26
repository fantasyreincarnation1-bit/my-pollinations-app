> Generate text, images, video, audio, realtime voice, and embeddings with a single API. OpenAI-compatible — use any OpenAI SDK by changing the base URL.

**Base URL:** `https://gen.pollinations.ai`

**Get your API key:** [enter.pollinations.ai](https://enter.pollinations.ai/keys)

**Integration guides:** [BYOP, CLI, MCP Server](/docs/guides)

## Quick Start

### Text (Python, OpenAI SDK)

```python
from openai import OpenAI
client = OpenAI(base_url="https://gen.pollinations.ai/v1", api_key="YOUR_API_KEY")
response = client.chat.completions.create(model="openai", messages=[{"role": "user", "content": "Hello!"}])
print(response.choices[0].message.content)
```

### Image (URL — no code needed)

```plaintext
https://gen.pollinations.ai/image/a%20cat%20in%20space?model=flux
```

### Audio (cURL)

```bash
curl "https://gen.pollinations.ai/audio/Hello%20world?voice=nova" \
  -H "Authorization: Bearer YOUR_API_KEY" -o speech.mp3
```

### 3D (cURL)

```bash
curl "https://gen.pollinations.ai/3d/no_prompt_for_trellis_needed?image=https://inferenceport.ai/img/trellis.jpg&model=trellis-2-low" \
  -H "Authorization: Bearer YOUR_API_KEY" -o model.glb
```

### Embeddings (OpenAI-compatible)

```bash
curl https://gen.pollinations.ai/v1/embeddings \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model":"openai-3-small","input":"Hello world","dimensions":512}'
```

See `GET /v1/models` for every text, image, audio, video, and embedding model available.

## Authentication

All generation requests require an API key from [enter.pollinations.ai](https://enter.pollinations.ai/keys). Model listing endpoints work without authentication.

| Type | Prefix | Use case | Rate limits | Description |
|------|--------|----------|-------------|-------------|
| Secret | `sk_` | Server-side apps | None | Personal developer key. Never expose in client-side code. |
| App Key (BYOP) | `pk_` | Client-side & Frontend apps | None | Publishable key used in the **BYOP (Bring Your Own Pollen)** flow to authorize users' balances. |

> **Note:** Publishable Keys (`pk_`) for direct client-side requests have been replaced by the **BYOP (Bring Your Own Pollen)** auth flow. Frontend applications should use the OAuth authorization-code flow with PKCE to obtain a temporary user-authorized secret key (`sk_`). The legacy fragment redirect and device flow remain supported.

Two ways to authenticate generation requests:

- Header: `Authorization: Bearer YOUR_API_KEY`
- Query param: `?key=YOUR_API_KEY`

For detailed integration guides on user-pays authorization, including OAuth discovery and token exchange, refer to the [Bring Your Own Pollen (BYOP) guide](https://github.com/pollinations/pollinations/blob/main/BRING_YOUR_OWN_POLLEN.md).

## Text Generation

Generate text responses using AI models. Fully compatible with the OpenAI Chat Completions API — use any OpenAI SDK by changing the base URL.

| Endpoint | Best for |
|----------|----------|
| `POST /v1/chat/completions` | Full OpenAI compatibility — streaming, tools, vision, structured outputs |
| `GET /text/{prompt}` | Quick prototyping — simple GET, returns plain text |

**Available models:** openai, openai-fast, gpt-oss, gpt-5.4, gpt-5.4-mini, openai-large, gpt-5.6-sol, gpt-5.6-terra, gpt-5.6-luna, mercury, qwen-coder, mistral-small-3.2, mistral, openai-audio, openai-audio-large, gemini-3-flash, gemini, gemini-flash-lite-3.5, gemini-fast, deepseek, gemma, gemma-4-31b, deepseek-pro, grok, grok-4-20-reasoning, grok-large, grok-4.5, gemini-search, gemini-search-fast, gemini-search-large, midijourney, midijourney-large, claude-fast, claude, claude-sonnet-5, claude-opus-4.6, claude-opus-4.7, claude-large, claude-fable-5, perplexity-fast, perplexity-high, perplexity, perplexity-reasoning, kimi, kimi-code, kimi-k3, laguna, longcat, mimo-v2.5, mimo-v2.5-pro, gemini-large, nova-fast, nova, glm, llama, llama-maverick, llama-scout, minimax-m2.7, minimax, muse-spark-1.1, mistral-large, qwen-coder-large, qwen-large, qwen3.7-max, qwen-vision, qwen-vision-pro, step-flash, step-3.5-flash, qwen-safety

### Prompt caching

On Gemini, Claude, and Nova models, a large static prompt prefix can be cached so repeat requests bill it at a fraction of the input rate. Mark the end of the static prefix with `cache_control` on a content block (not on the message); everything before the marker must be byte-identical across requests, everything dynamic goes after. The first request creates the cache (`usage` reports `cache_creation_input_tokens`); repeat requests within the TTL report `prompt_tokens_details.cached_tokens` at the discounted rate.

```json
{
  "model": "gemini-fast",
  "messages": [
    {
      "role": "system",
      "content": [
        {
          "type": "text",
          "text": "<large static prompt>",
          "cache_control": { "type": "ephemeral" }
        }
      ]
    },
    { "role": "user", "content": "<dynamic message>" }
  ]
}
```

**Gemini** — the prefix must be at least ~2,048 tokens (~4,096 on Gemini 3 models). Requests with tools are not cached — including built-in tools, so `gemini`, `gemini-3-flash`, `gemini-large`, and the search variants only cache when tools are disabled (`"tools": []`) or a JSON `response_format` is set; `gemini-fast` and `gemini-flash-lite-3.5` cache by default. Cache creates bill at the standard input rate plus a storage fee for the 1-hour TTL ($1 per 1M cached tokens on Flash models, $4.50 on Pro); hits bill at ~10% of input. The storage fee means caching pays off only when the prefix is reused often — roughly a dozen reuses per hour on the cheapest models.

**Claude** — all Claude models cache. The prefix must be at least 4,096 tokens (1,024 on `claude` and `claude-fable-5`); tools are fine. Cache creates bill at 1.25× the input rate (no storage fee); hits bill at 10% of input. The cache lives ~5 minutes, refreshed on each hit.

**Nova** — `nova` and `nova-fast` cache. The prefix must be at least ~1,000 tokens (up to 20K tokens cacheable). Cache creates are free; hits bill at 25% of input. ~5-minute TTL.

## Image Generation

Generate images from text prompts via a simple GET request. Returns JPEG, PNG, or SVG depending on the selected model.

```
https://gen.pollinations.ai/image/a%20cat%20in%20space?model=flux
```

**Available models:** sana, kontext, nanobanana, nanobanana-2, nanobanana-2-lite, nanobanana-pro, seedream5, seedream5-pro, seedream, seedream-pro, ideogram-v4-turbo, ideogram-v4-balanced, ideogram-v4-quality, gptimage, gptimage-large, gpt-image-2, flux, zimage, wan-image, wan-image-pro, qwen-image, grok-imagine, grok-imagine-pro, recraft-v4.1-vector, klein, p-image, p-image-edit, nova-canvas

### Community image models

Community image models use an owner/model id and support text-to-image generation through `/image/{prompt}` and `/v1/images/generations`. For the OpenAI-compatible endpoint, use `response_format: "b64_json"`. URL responses, reference images, and `/v1/images/edits` are not supported for community models yet. See `/image/models` for the live model list and supported endpoints.

## Video Generation

Generate videos from text prompts or reference images. Returns MP4.

```
https://gen.pollinations.ai/video/sunset%20timelapse?model=veo&duration=4
```

**Available models:** veo, veo-1080p, seedance-pro, seedance-2.0, wan, wan-fast, wan-pro, wan-pro-1080p, grok-video-pro, happyhorse-1.1, p-video-720p, p-video-1080p, nova-reel

## Realtime Voice

OpenAI-compatible Realtime WebSocket proxy for voice and multimodal sessions.

| Endpoint | Description |
|----------|-------------|
| `GET /v1/realtime` | WebSocket Realtime session (`model=gpt-realtime-2.1`) |

Requires an API key with positive balance. Server clients can use `Authorization: Bearer <key>`; browser WebSocket clients can use `?key=pk_...`.

The WebSocket proxy aggregates observed `response.done` usage and settles one billing event when the session closes. Input transcription sessions are not supported yet.

Events sent and received over the socket use the OpenAI Realtime protocol unchanged. See OpenAI's [Realtime WebSocket events guide](https://developers.openai.com/api/docs/guides/realtime-websocket#sending-and-receiving-events).

```js
import WebSocket from "ws";

// Server: Bearer auth. Browser: append `&key=pk_...` instead (headers aren't settable).
const ws = new WebSocket(
    "wss://gen.pollinations.ai/v1/realtime?model=gpt-realtime-2.1",
    { headers: { Authorization: `Bearer ${process.env.POLLINATIONS_API_KEY}` } },
);

ws.on("open", () => ws.send(JSON.stringify({
    type: "session.update",
    session: { type: "realtime", instructions: "Be concise." },
})));
ws.on("message", (m) => console.log(JSON.parse(m.toString())));
```

**Browser audio:** play the model's audio through an `<audio>` element (e.g. a Web Audio `MediaStreamDestination` set as the element's `srcObject`), not straight to the Web Audio output. The browser only uses audio-element output as the echo-cancellation reference, so without it the mic re-captures the model's voice and it starts replying to itself. The WebRTC transport handles this automatically; on the WebSocket transport it's the client's responsibility.

**Realtime models:** gpt-realtime-2.1, gpt-realtime-2

## 3D Generation

Generate 3D models from text prompts and images via a simple GET request.
Returns glTF Binary in GLB format. Depending on the model, certain models
ignore text inputs — any text prompt passed to the Trellis 2 family will be
ignored; only the image URL is used.

https://gen.pollinations.ai/3d/no_prompt_for_trellis_needed?model=trellis-2-low&key=YOUR_KEY_HERE&image=IMAGE_URL_HERE

**Available models:** trellis-2-low, trellis-2-medium, trellis-2-high, hyper3d-rodin

> **Note:** `hyper3d-rodin` requires Paid Pollen. `trellis-2-low` (the default),
> `trellis-2-medium`, and `trellis-2-high` work with Quest Pollen.

## Audio Generation

Text-to-speech, music generation, and audio transcription.

| Endpoint | Description |
|----------|-------------|
| `GET /audio/{text}` | Simple URL-based TTS or music generation |
| `POST /v1/audio/speech` | OpenAI-compatible TTS |
| `POST /v1/audio/transcriptions` | Speech-to-text transcription |

**Audio models:** elevenlabs, elevenflash, eleven-multilingual-v2, elevenmusic, lyria-3-clip, eleven-sfx, whisper, scribe, universal-2, universal-3-pro, stable-audio-3-medium, stable-audio-3-large, qwen-tts, qwen-tts-instruct, csm-1b

**Available voices:** alloy, echo, fable, onyx, nova, shimmer, ash, ballad, coral, sage, verse, rachel, domi, bella, elli, charlotte, dorothy, sarah, emily, lily, matilda, adam, antoni, arnold, josh, sam, daniel, charlie, james, fin, callum, liam, george, brian, bill

## Embeddings

Generate vector embeddings with an OpenAI-compatible response format.

| Endpoint | Description |
|----------|-------------|
| `POST /v1/embeddings` | OpenAI-compatible embeddings endpoint |
| `GET /embeddings/models` | Embedding models with pricing and modalities |

`gemini-2` supports text, image, audio, and video inputs. `cohere-embed-v4` supports text and one image per input. The OpenAI and Qwen embedding models are text-only.

String batch input supports up to 32 items. For retrieval, use `task_type` with Gemini text input (it is converted to the recommended prompt instruction) or `input_type` (`query` or `document`) with Cohere. Dimensions are model-specific: Cohere supports 256, 512, 1024, or 1536; `openai-3-small` supports up to 1536; `gemini-2` and `openai-3-large` support up to 3072; `qwen3-embedding-8b` supports up to 4096.

Gemini task instructions count toward prompt token usage. Cohere requests containing an image expose one combined usage count, so any accompanying text is billed at the image-input rate.

**Gemini GA migration:** `gemini-2` now uses the GA embedding space. Do not mix preview-era and GA vectors; re-embed stored `gemini-2` data before comparing it with new results.

**Embedding models:** gemini-2, openai-3-small, openai-3-large, cohere-embed-v4, qwen3-embedding-8b

## Models

Discover available models with pricing, capabilities, and metadata. No authentication required.

| Endpoint | Returns |
|----------|---------|
| `GET /models` | All models with pricing, capabilities, and metadata |
| `GET /v1/models` | All models in OpenAI-compatible format (`{object: "list", data: [...]}`) |
| `GET /text/models` | Text models with pricing, context window, tool support |
| `GET /image/models` | Image & video models with capabilities and pricing |
| `GET /audio/models` | Audio models with supported voices |
| `GET /embeddings/models` | Embedding models with supported modalities |
| `GET /3d/models` | 3D Generation models with supported modalities |

Rich model endpoints include `capabilities` for agentic/model traits:
`tool_calling`, `reasoning`, `web_search`, and `code_execution`.
Modalities, video frame controls, voices, and context length remain separate
structured fields.

## Community Models (Alpha)

Community models are user-owned, OpenAI-compatible text or image-generation endpoints proxied through `gen.pollinations.ai` under an `owner/model` id (e.g. `Spit-fires/LFM2.5-230M`). Text providers serve `/v1/chat/completions`; image providers serve `/v1/images/generations`. Any signed-in user can register a private, owner-only model. Publishing it in the public model catalog requires allowlist approval while the program is in alpha.

**Alpha stage**
- Inclusion is fairly permissive for now; expect that to tighten before official launch.
- Text and image generation models are supported now; audio and other modalities are planned next.
- Community image models are text-to-image only and are exposed through `/image/{prompt}` and `/v1/images/generations`. OpenAI-compatible calls must use `response_format: "b64_json"`; reference images and `/v1/images/edits` are not supported yet.

**Pricing**
- Owners set prices when publishing; blank or zero means free.
- Text models are priced per token (entered per 1M tokens) for each usage bucket the endpoint reports.
- Image models bill in one of two modes, detected when the endpoint is tested at registration: endpoints that return OpenAI image token usage are priced per 1M tokens; all others charge a fixed price per generated image.

**Payouts**
- Owners currently earn 75% of the pollen spent on their model.
- Payouts are like-for-like: a request paid with paid pollen pays the owner in paid pollen; a request paid with quest pollen pays the owner in quest pollen. Quest pollen can't be cashed out — it can only be spent on non-paid models.
- Owners will be able to switch their model to paid-only.
- Dollar payouts are planned but not available yet (legal/compliance work in progress).
- Expect a trial period where pollen accumulates but can't be cashed out yet — this will likely start manually, inviting owners once they cross a pollen threshold.

**Policing & safety**
- Community models do **not** run on Pollinations infrastructure — they run on the owner's own backend. Don't send API keys or other sensitive data through them.
- A safety feature that auto-redacts private info before it's sent to community models is planned, likely on by default with an opt-out.
- Owners and users are encouraged to test each other's models — self-policing keeps the ecosystem honest.
- Models can be pulled (and repeat offenders potentially blocked) for instability or suspected abuse — e.g. silently changing prices or serving a different model than advertised.

**Automated health monitoring**
- An automated monitor currently checks community text models for error rate and latency. Models with sustained failures get deactivated automatically — no human involvement needed for that direction. Community image models are not monitored yet.
- Reactivating a deactivated model is manual and owner-only, from the dashboard. There's no auto-reactivation, so if your model was turned off, fix the underlying issue before reactivating it, or it may just fail again.
- Check your model's live health — request counts, success rate, errors, and latency — at [model-monitor.pollinations.ai/debug](https://model-monitor.pollinations.ai/debug).

To request account-level permission to publish community models, submit a [publisher allowlist request](https://github.com/pollinations/pollinations/issues/new?template=community-model-allowlist.yml). The form does not register individual models. Private models can be registered and called without approval under **Models → My Models** at [enter.pollinations.ai](https://enter.pollinations.ai); fetching upstream models, testing the upstream endpoint, and publishing require approval. Registration and management are also documented under the Account section of this reference. The dashboard and Account API support text and image models; the [CLI](/docs/guides/cli) (`polli my-models`) currently supports text models only.

## Media Storage

Upload images, audio, and video and get back a unique id and URL. Each upload gets its own id (re-uploading the same bytes yields a new one).

Base URL: https://media.pollinations.ai

| Endpoint | Description |
|----------|-------------|
| `POST /upload` | Upload a file, receive a unique media URL |
| `GET /{id}` | Retrieve a previously uploaded file |
| `GET /{id}/metadata` | Get file metadata as JSON |
| `GET /media?tag={tag}` | List the public gallery for a tag (no auth) |
| `DELETE /media/{id}` | Delete a published item you own (secret `sk_` key) |

Upload requires an API key; retrieval is public. The decoded/file-size limit is 100MB for both upload formats. Files use a 30-day lifecycle from upload or the latest refresh. Retrieving the file body refreshes that lifecycle only when the object is at least 15 days old; metadata and HEAD requests do not refresh it. Two upload formats are accepted:

Multipart form (browsers, files on disk):

```bash
curl -X POST "https://media.pollinations.ai/upload" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F file=@path/to/image.png
```

Base64 JSON (programmatic callers that already hold the bytes):

```bash
curl -X POST "https://media.pollinations.ai/upload" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"data": "<base64-or-data-uri>", "contentType": "image/png", "name": "image.png"}'
```

**Tags publish (alpha).** An optional `tags` field (comma-separated string, or a JSON array in the JSON format) publishes the upload into each tag's public gallery, where anyone can list it via `GET /media?tag={tag}`. Untagged uploads stay unlisted — reachable only by their unguessable id URL. Full endpoint reference: https://media.pollinations.ai/openapi.json

## Account

Self-service endpoints for the authenticated user. All endpoints require authentication (API key or session). API keys need the relevant `account:<scope>` permission. Base path: `/account`.

`account:usage` is the read-only account-state scope for balances, usage, quests, and earnings. `account:keys` manages keys and, where enabled, my-models. These permissions are independent; request both when a client needs both. Newly created child keys cannot receive `account:keys` through this API.

| Endpoint | Description |
|----------|-------------|
| `GET /account/profile` | GitHub username, image, and community model access |
| `GET /account/balance` | Current pollen balance |
| `GET /account/quests` | Read-only quest status |
| `GET /account/usage` | Per-request usage history with costs |
| `GET /account/usage/daily` | Daily aggregated usage for dashboards |
| `/account/my-models` | Private community model registration and allowlisted public publishing |
| `GET /account/key` | API key validity, type, and permissions |

### GET /account/profile

Returns user profile. `githubUsername`, `image`, and `communityEndpointsAllowed` are always included. `name` and `email` are included only when the API key has `account:profile`.

### GET /account/balance

Returns rem# my-pollinations-app
