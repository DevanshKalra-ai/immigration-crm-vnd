# Immigration CRM — Strict AI Lead Scoring (n8n)

Plug-and-play n8n workflow that scores student-visa applicants as Hot / Warm / Cold using Google Gemini with strict hard-cap logic.

**Import URL:** https://devanshkalra-ai.github.io/immigration-crm-vnd/workflow.json
**Live demo form:** https://devanshkalra-ai.github.io/immigration-crm-vnd/apply.html

---

## Import into n8n (2 minutes)

**Method A — From file**
1. Save [`workflow.json`](./workflow.json) locally
2. n8n web → top-right menu → **Import from File** → select the file

**Method B — From URL** (n8n Desktop / self-hosted)
1. n8n web → top-right menu → **Import from URL**
2. Paste: `https://devanshkalra-ai.github.io/immigration-crm-vnd/workflow.json`

## Setup (one-time)

1. **Get a free Gemini key** at https://aistudio.google.com/apikey
2. In the imported workflow, click the **Gemini AI** node
3. Credential dropdown → **Create new credential** → **Query Auth**
   - Name: `key`
   - Value: *(paste your Gemini API key)*
   - Save
4. Toggle workflow **Active** (top right)
5. Copy the **Production URL** from the Webhook node

## Use

POST JSON to the webhook URL:

```json
{
  "full_name": "Aarav Sharma",
  "email": "aarav@example.com",
  "age": 23,
  "nationality": "Indian",
  "target_country": "Canada",
  "program": "Masters",
  "field_of_study": "Computer Science",
  "education_level": "Bachelor 8.2 CGPA",
  "english_proficiency": "IELTS 7.5",
  "budget": "$45000",
  "has_sponsorship": "parents + partial scholarship",
  "visa_rejected": "No",
  "motivation": "..."
}
```

Response (~5 seconds):

```json
{
  "success": true,
  "percentage": 78,
  "label": "Warm",
  "summary": "Strong English and academics but motivation lacks specific university and program details, preventing Hot tier.",
  "key_factors": [
    "Good academic record (8.2 CGPA) and IELTS 7.5",
    "Partial (not full) scholarship",
    "Motivation lacks specific university details"
  ]
}
```

## Scoring Logic

### Hard caps (applied before band assignment)
| Trigger | Max score |
|---|---|
| Any prior visa rejection | 60 (cannot be Hot) |
| No English test or below floor | 35 |
| Budget below country annual floor | 50 |
| Missing 3+ critical fields | 40 |

### Country annual cost floors (USD)
UK $35k · US $45k · Canada $25k · Australia $35k · Germany $12k · NZ $25k · Ireland $30k · France $18k · Netherlands $25k

### English floors
- Undergraduate: IELTS 6.0 / TOEFL 70
- Masters: IELTS 6.5 / TOEFL 80
- PhD: IELTS 7.0 / TOEFL 90

### Penalties
- Motivation under 15 words: −25
- Motivation 15–29 words: −10
- Field mismatched with prior academics: −15 (judged by Gemini)

### Tier requirements
- **Hot (85–100)** — *all of:* IELTS ≥ 7.5, funding ≥ 1.5× floor or full scholarship, zero rejections, specific motivation, aligned field, strong academics
- **Warm (55–84)** — meets minimums, profile coherent
- **Cold (0–54)** — any hard cap triggered

## Adding storage (optional)

The workflow ends with the response. Drop any storage node *after* `Parse AI Response`:
- **Google Sheets** — append row
- **Airtable / Notion** — create record
- **Postgres / MySQL** — insert row
- **HTTP Request** — POST to your own backend

All 16 form fields plus `ai_percentage`, `ai_label`, `ai_summary`, `ai_key_factors` are available on `$json`.

## Pipeline

```
Webhook → Gemini 2.5 Flash-Lite → Parse + Hard-cap enforcement → Respond
```

Latency: ~5 seconds end-to-end.

## License

MIT. Built during VND Global internship — see report.
