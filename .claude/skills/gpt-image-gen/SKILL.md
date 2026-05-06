---
name: "gpt-image-gen"
description: "Generate an image via OpenAI's Images API and save it to disk as a PNG. Use whenever any agent needs to produce an image from a text prompt — yuval is the primary caller, but any agent may use this skill directly. Requires OPENAI_API_KEY in the project's .env. Returns the absolute output path on success; halts with a clear error on missing key, API failure, or empty file."
---

# gpt-image-gen

Shared wrapper around `POST https://api.openai.com/v1/images/generations`. Centralizes auth, the curl recipe, base64 decoding (with a Python fallback for environments where `jq` isn't on PATH — e.g. Git Bash on Windows), and post-write verification. Every agent that produces a PNG goes through this skill so the mechanics live in one place.

## When to invoke

Any time an agent needs to turn a text prompt into a PNG file on disk.

## Prerequisites

- `OPENAI_API_KEY` must be set in `.env` at the project root.
- The output directory must already exist — this skill writes to a path the caller provides; it does not create directories.
- One of: `jq` on PATH, **or** `python` on PATH (fallback). At least one must be available to decode the base64 response.

## Inputs (caller-supplied)

| Name | Required | Default | Notes |
|---|---|---|---|
| `prompt` | yes | — | Free-text prompt. English works best with the model. |
| `output_path` | yes | — | Absolute or repo-relative path ending in `.png`. Parent directory must already exist. |
| `size` | no | `1024x1024` | One of the API-supported sizes. |
| `quality` | no | `medium` | One of `low` / `medium` / `high`. |

## Procedure

Run this as a single Bash invocation. Substitute `<PROMPT>` and `<OUTPUT_PATH>` with the caller's values. Quote the prompt safely (escape any embedded `"`).

```bash
# 1. Load API key from .env
set -a; source .env; set +a
if [ -z "$OPENAI_API_KEY" ]; then
  echo "ERROR: OPENAI_API_KEY is not set in .env" >&2
  exit 1
fi

PROMPT='<PROMPT>'
OUTPUT_PATH='<OUTPUT_PATH>'
SIZE='1024x1024'
QUALITY='medium'

# 2. Build the JSON payload safely (lets jq/python escape quotes if available;
#    fall back to a heredoc when neither is, but that path is rare).
PAYLOAD=$(jq -n \
  --arg model "gpt-image-2" \
  --arg prompt "$PROMPT" \
  --arg size "$SIZE" \
  --arg quality "$QUALITY" \
  '{model:$model, prompt:$prompt, size:$size, quality:$quality, output_format:"png"}' 2>/dev/null) \
  || PAYLOAD=$(python -c "import json,sys,os; print(json.dumps({'model':'gpt-image-2','prompt':os.environ['PROMPT'],'size':os.environ['SIZE'],'quality':os.environ['QUALITY'],'output_format':'png'}))")

# 3. Call the API
RESPONSE_FILE=$(mktemp -t openai_image_response.XXXXXX.json)
HTTP_CODE=$(curl -sS -w "%{http_code}" -o "$RESPONSE_FILE" \
  -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "$PAYLOAD")

if [ "$HTTP_CODE" != "200" ]; then
  echo "ERROR: OpenAI API returned HTTP $HTTP_CODE" >&2
  cat "$RESPONSE_FILE" >&2
  exit 1
fi

# 4. Decode base64 → PNG. Prefer jq+base64; fall back to Python.
if command -v jq >/dev/null 2>&1; then
  jq -r '.data[0].b64_json' "$RESPONSE_FILE" | base64 --decode > "$OUTPUT_PATH"
else
  python -c "import json,base64,sys; d=json.load(open(sys.argv[1])); open(sys.argv[2],'wb').write(base64.b64decode(d['data'][0]['b64_json']))" "$RESPONSE_FILE" "$OUTPUT_PATH"
fi

# 5. Verify the file exists and is non-empty
if [ ! -s "$OUTPUT_PATH" ]; then
  echo "ERROR: output file is empty or missing: $OUTPUT_PATH" >&2
  echo "API response was:" >&2
  cat "$RESPONSE_FILE" >&2
  exit 1
fi

rm -f "$RESPONSE_FILE"
echo "OK: $OUTPUT_PATH"
```

## Reference: minimal curl (for documentation / debugging)

The bare-bones request, exactly per the project spec:

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

## Python decode fallback (standalone)

For environments where `jq` is not installed (Git Bash on Windows often lacks it). Save the API response JSON to a file first, then:

```bash
python -c "import json,base64,sys; d=json.load(open(sys.argv[1])); open(sys.argv[2],'wb').write(base64.b64decode(d['data'][0]['b64_json']))" response.json output.png
```

## Error modes

| Symptom | Likely cause | Fix |
|---|---|---|
| `OPENAI_API_KEY is not set` | `.env` missing the key | Add `OPENAI_API_KEY=sk-...` to `.env` |
| HTTP 401 | Invalid / revoked key | Check the key at platform.openai.com |
| HTTP 400 with `"model"` in error message | Model name not available on this account | Confirm `gpt-image-2` access; otherwise update the skill |
| HTTP 429 | Rate limit | Wait and retry; consider lower `quality` |
| Empty `.png` (0 bytes) | Decode step failed | Re-run with `jq`/`python` available; inspect response file |
| Non-PNG bytes in output | Wrong `output_format` | Skill hard-codes `png`; if changed, mirror the file extension |

## Output contract

On success, the skill writes a single PNG to the caller-provided `output_path` and prints `OK: <output_path>`. The caller is responsible for:

- Creating the parent directory before invoking.
- Handling any sidecar metadata (e.g. saving the prompt next to the file).
- Re-running on transient errors.
