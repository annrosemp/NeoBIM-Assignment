# 🏗️ Ökobaudat Material Matcher

An AI-powered pipeline that automatically matches building elements to Environmental Product Declarations (EPDs) from the [Ökobaudat](https://www.oekobaudat.de/) database, grounded in German and EU construction standards.

---

## What it does

Given a building element (e.g. an external wall, flat roof, or floor slab), the pipeline:

1. **Decomposes** the element into its standard material layers using Claude + live web search against German/EU norms (DIN, EN ISO, GEG, DGNB)
2. **Searches** the Ökobaudat REST API for matching EPDs per layer
3. **Ranks** and selects the best EPD match per layer using Claude as an LCA expert
4. **Verifies** the complete assembly for compliance with German construction standards
5. **Outputs** a structured JSON file + a human-readable Markdown report

---

## Pipeline architecture

```
User input (element dict)
        │
        ▼
┌─────────────────────────────────┐
│  STEP 1: Layer Decomposition    │
│  Claude + web_search tool       │
│  → DIN / EN ISO / GEG norms     │
└────────────────┬────────────────┘
                 │ layers[]
                 ▼
┌─────────────────────────────────┐
│  STEP 2: Ökobaudat API Search   │
│  REST GET /datastocks/.../      │
│  processes?search=true&name=... │
│  → candidate EPDs per layer     │
└────────────────┬────────────────┘
                 │ candidates{}
                 ▼
┌─────────────────────────────────┐
│  STEP 3: EPD Ranking            │
│  Claude as LCA expert           │
│  → best_uuid, confidence,       │
│    reasoning per layer          │
└────────────────┬────────────────┘
                 │ matched[]
                 ▼
┌─────────────────────────────────┐
│  STEP 4: Standards Verification │
│  Claude as building physicist   │
│  → COMPLIANT / PARTIAL /        │
│    NON_COMPLIANT + score/100    │
└────────────────┬────────────────┘
                 │
                 ▼
        JSON + Markdown report
```

---

## Quickstart (Google Colab)

### 1. Add your Anthropic API key to Colab Secrets

In Colab: **🔑 Secrets (left sidebar) → Add new secret**

- Name: `ANTHROPIC_API_KEY`
- Value: your key from [console.anthropic.com](https://console.anthropic.com/settings/apikeys)

> ⚠️ Never paste your API key directly into a notebook cell. Always use Colab Secrets or environment variables.

### 2. Open the notebook

Upload `okobaudat_pipeline.ipynb` to Google Colab or open it directly from GitHub via the badge below.

### 3. Run cells in order

| Cell | What it does |
|------|-------------|
| **Cell 1** | Installs `anthropic`, `requests`, `rich` |
| **Cell 2** | Loads API key from Colab Secrets into environment |
| **Cell 3** | Defines the full pipeline (no execution yet) |
| **Cell 4** | Runs the pipeline — edit the `element` dict here |

### 4. Edit the element definition in Cell 4

```python
element = {
    "name":        "External Wall",
    "elementType": "Wall",
    "thickness":   300,        # total thickness in mm
}
```

Supported `elementType` values include: `Wall`, `Roof`, `Floor`, `Basement Wall`, or any descriptive string — Claude infers the appropriate standards automatically.

### 5. Collect outputs

After the run completes, `okobaudat_result.json` is automatically downloaded. It contains the full structured output including all layer data, EPD matches, and the compliance report.

---

## Output format

### JSON (`okobaudat_result.json`)

```json
{
  "element": { "name": "...", "elementType": "...", "thickness": 200 },
  "generated_at": "2026-05-21T10:00:00",
  "datastock": "OEKOBAUDAT 2021-II",
  "decomposition": {
    "layers": [
      {
        "name": "Concrete Deck",
        "thickness_mm": 120,
        "description": "...",
        "standard_ref": "DIN 1045",
        "search_keywords": ["reinforced concrete slab", "Stahlbeton"]
      }
    ],
    "references": [{ "code": "DIN 4108-2", "title": "...", "url": "..." }],
    "total_thickness_mm": 200,
    "notes": "..."
  },
  "matched_layers": [
    {
      "layer": { ... },
      "best_match": {
        "best_uuid": "abc-123",
        "best_name": "Normalbeton C25/30",
        "confidence": "high",
        "reasoning": "...",
        "url": "https://www.oekobaudat.de/..."
      },
      "candidates": [ ... ]
    }
  ],
  "verification": {
    "overall_verdict": "COMPLIANT",
    "overall_score": 82,
    "summary": "...",
    "layer_checks": [ ... ],
    "recommendations": [ ... ],
    "standards_checked": ["DIN 4108-2", "GEG 2024"]
  },
  "report_markdown": "# Ökobaudat Material Matching Report\n..."
}
```

### Console output

The pipeline prints a live progress log using `rich` (coloured ✔/⚠/✖ indicators per step) and renders the final Markdown report inline.

---

## Standards referenced

| Standard | Scope |
|----------|-------|
| **DIN 4108-2** | Minimum thermal insulation requirements |
| **DIN 4108-3** | Moisture protection / vapour barriers |
| **DIN 18531** | Waterproofing of flat roofs |
| **DIN 18195** | Waterproofing of buildings (below ground) |
| **DIN 18550** | External render systems |
| **DIN 1045** | Concrete and reinforced concrete structures |
| **EN ISO 6946** | Thermal resistance and transmittance calculation |
| **GEG 2024** | Gebäudeenergiegesetz — German Building Energy Act |
| **DGNB** | German Sustainable Building Council criteria |

---

## Assumptions and limitations

- **Datastock fixed to OEKOBAUDAT 2021-II** (`cd2bda71-760b-4fcc-8a0b-3877c10000a8`). Newer datastocks may require updating `DATASTOCK_UUID`.
- **Layer thickness is distributed proportionally** to meet the total element thickness. Where a norm mandates a minimum thickness (e.g. minimum 80 mm insulation under GEG), Claude respects that constraint first and distributes remaining thickness across other layers.
- **EPD search is name-based** — the Ökobaudat API does not support semantic search, so match quality depends on keyword alignment. The pipeline tries the primary layer name, then `search_keywords` generated by Claude, then a single-word fallback.
- **Compliance verification is AI-generated** and is intended as a first-pass review aid, not a substitute for a certified building physicist or official code check.
- **The Ökobaudat API is public and requires no authentication**, but may be rate-limited or unreachable from certain network environments (e.g. sandboxed cloud runtimes).
- **Model**: defaults to `claude-opus-4-6`. Change the `MODEL` constant in Cell 3 to use a different model (e.g. `claude-sonnet-4-6` for faster/cheaper runs, `claude-haiku-4-5-20251001` for lightweight use).

---

## Known issues and fixes

### `'str' object has no attribute 'get'`
The Ökobaudat API inconsistently returns the `name` field as either a plain string or a localised list `[{"lang": "en", "value": "..."}]`. This notebook includes a fix that handles both cases:

```python
raw_name = p.get("name")
if isinstance(raw_name, list):
    name_val = raw_name[0].get("value", p.get("uuid", "")) if raw_name else p.get("uuid", "")
elif isinstance(raw_name, str):
    name_val = raw_name
else:
    name_val = p.get("uuid", "")
```

### Rate limit errors (429)
The Anthropic free tier is limited to 30,000 input tokens/minute. Step 1 uses the web search tool which is token-heavy. If you hit a 429, wait ~60 seconds and re-run Cell 4. Upgrading to a paid tier resolves this for production use.

### 403 Forbidden from Ökobaudat
The Ökobaudat API blocks requests from certain cloud IP ranges (including some Colab regions). If all Step 2 searches return 403, try running the notebook locally or from a different network.

---

## Security

- **API keys are never hardcoded.** The notebook reads `ANTHROPIC_API_KEY` exclusively from Colab Secrets / environment variables.
- **Do not commit** `okobaudat_result.json` to version control if it contains sensitive project data.
- Add the following to your `.gitignore`:

```
okobaudat_result.json
*.json
.env
```

---

## Requirements

- Python 3.10+
- `anthropic >= 0.25`
- `requests`
- `rich` (optional, for coloured output)
- An Anthropic API key with access to the `web_search` tool (requires at least Build tier)

---

## License

MIT — see `LICENSE` for details.

---

## Acknowledgements

- [Ökobaudat](https://www.oekobaudat.de/) — German national EPD database, maintained by the Federal Ministry for Housing, Urban Development and Building
- [Anthropic](https://www.anthropic.com/) — Claude API and web search tool
