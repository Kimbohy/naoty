# 🎵 SolfaHarmony

> Automatic harmonic annotation for Tonic Sol-fa scores — upload a PDF, get chord symbols back.

![Phase](https://img.shields.io/badge/phase-1%20%E2%80%94%20Web%20App-blue)
![Python](https://img.shields.io/badge/python-3.13%2B-yellow)
![React](https://img.shields.io/badge/react-18-61dafb)
![License](https://img.shields.io/badge/license-MIT-green)

---

## What it does

SolfaHarmony reads a **Tonic Sol-fa PDF** (the letter-based notation system used in choral traditions — `d r m f s l t`) and overlays **Roman numeral chord symbols** directly on the score.

No OCR needed — it reads the text coordinates directly from the PDF, aligns the four SATB voices beat by beat, identifies each chord, and writes the annotations back onto the file.

**Input → Output:**

```
Original PDF (sol-fa, no harmony)     Annotated PDF
┌─────────────────────────────┐       ┌─────────────────────────────┐
│  d . d : m . m  r : d  ...  │       │  I              V   I  ...  │
│  d . d : m . m  r : d  ...  │  →    │  d . d : m . m  r : d  ...  │
│  s,.s, : d . d  t,: s, ...  │       │  s,.s, : d . d  t,: s, ...  │
│  m . m : s . s  f : m  ...  │       │  m . m : s . s  f : m  ...  │
│  d . d : d . d  s,: d  ...  │       │  d . d : d . d  s,: d  ...  │
└─────────────────────────────┘       └─────────────────────────────┘
```

---

## Tech stack

| Layer            | Technology                           |
| ---------------- | ------------------------------------ |
| Frontend         | React 18 + TypeScript + Tailwind CSS |
| PDF viewer       | PDF.js (Mozilla)                     |
| Backend          | Python 3.13 / FastAPI                |
| PDF extraction   | pdfplumber                           |
| PDF annotation   | PyMuPDF (fitz)                       |
| Frontend hosting | Vercel / Netlify                     |
| Backend hosting  | Railway / Render                     |

---

## Project structure

```
solfaharmony/
├── backend/
│   ├── main.py                  # FastAPI app & endpoints
│   ├── requirements.txt
│   └── modules/
│       ├── pdf_extractor.py     # Text + coordinate extraction
│       ├── line_classifier.py   # Classify rows: header / notes / lyrics
│       ├── system_builder.py    # Group voice lines into SATB systems
│       ├── beat_aligner.py      # Align 4 voices by x-position
│       ├── chord_detector.py    # Match degree sets to chord templates
│       └── pdf_annotator.py     # Write annotations onto the PDF
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/
│   │   │   ├── DropZone.tsx     # PDF upload area
│   │   │   ├── PdfViewer.tsx    # Side-by-side PDF display
│   │   │   └── ConfigPanel.tsx  # Key, detail level, color options
│   │   ├── pages/
│   │   │   ├── Home.tsx
│   │   │   └── Result.tsx
│   │   └── api/
│   │       └── client.ts        # FastAPI communication
│   ├── package.json
│   └── tailwind.config.ts
└── README.md
```

---

## Getting started

### Prerequisites

- Python 3.13+
- Node.js 18+
- pip

### Backend

```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
```

The API will be available at `http://localhost:8000`.  
Interactive docs: `http://localhost:8000/docs`

### Frontend

```bash
cd frontend
npm install
npm run dev
```

The app will be available at `http://localhost:5173`.

### Environment variables

Create a `.env` file in `frontend/`:

```env
VITE_API_URL=http://localhost:8000
```

Create a `.env` file in `backend/`:

```env
ALLOWED_ORIGINS=http://localhost:5173
MAX_FILE_SIZE_MB=10
```

---

## API

### `POST /analyze`

Accepts a PDF file and analysis parameters, returns an annotated PDF.

**Request** — `multipart/form-data`

| Field              | Type                  | Required | Description                                    |
| ------------------ | --------------------- | -------- | ---------------------------------------------- |
| `file`             | PDF file              | ✅       | The sol-fa score to annotate                   |
| `key`              | string                | ❌       | Override detected key (e.g. `"F"`, `"G"`)      |
| `detail_level`     | `basic` \| `detailed` | ❌       | Include inversions and 7ths (default: `basic`) |
| `show_uncertain`   | boolean               | ❌       | Show `?` chords (default: `false`)             |
| `annotation_color` | hex string            | ❌       | Color for chord labels (default: `"#cc0000"`)  |

**Response** — `application/pdf`

The annotated PDF as binary.

---

### `POST /preview`

Same input as `/analyze`, but returns a JSON chord analysis instead of a PDF. Useful for inspecting the detected chords before generating the file.

**Response**

```json
{
  "key": "F",
  "meter": "2/4",
  "systems": [
    {
      "index": 1,
      "chords": [
        {
          "beat_x": 16,
          "degrees": ["d", "m", "s"],
          "chord": "I",
          "certain": true
        },
        {
          "beat_x": 144,
          "degrees": ["s", "t", "r", "f"],
          "chord": "V7",
          "certain": true
        },
        {
          "beat_x": 200,
          "degrees": ["d", "m", "s"],
          "chord": "I",
          "certain": true
        }
      ]
    }
  ]
}
```

---

### `GET /health`

Returns `200 OK` when the server is running.

---

## How the analysis works

1. **Extract** — `pdfplumber` reads every text token from the PDF with its exact x/y coordinates. No OCR involved; the PDF must have selectable text.

2. **Classify** — rows are grouped by vertical position, then labelled as system headers (`Dô dia F 2/4`), note lines (`d . m : r . d …`), or lyric lines.

3. **Build systems** — consecutive groups of 4 note lines are assembled into SATB systems, each retaining its y-position on the page.

4. **Align beats** — within each system, tokens from all 4 voices that share the same x-position (±10 pts tolerance) are grouped into a single beat. Bar lines `|` serve as absolute anchors.

5. **Identify chords** — the set of scale degrees at each beat is matched against a template library (I, ii, iii, IV, V, V⁷, vi, vii°, V/V, It⁶ …). Repeated consecutive chords are collapsed.

6. **Annotate** — PyMuPDF writes the chord labels at the correct x position, in the free space above each system, without touching the original content.

---

## Supported sol-fa conventions

| Symbol          | Meaning                                        |
| --------------- | ---------------------------------------------- |
| `d r m f s l t` | Scale degrees 1–7                              |
| `d, r, m,` …    | Lower octave (comma suffix)                    |
| `d' r' m'` …    | Upper octave (apostrophe suffix)               |
| `fi si li ta`   | Chromatic alterations (raised/lowered degrees) |
| `-`             | Held note or rest                              |
| `.`             | Continuation dot (half-value extension)        |
| `\|`            | Bar line                                       |
| `:`             | Beat divider                                   |

---

## Known limitations

- **Scanned PDFs are not supported.** The PDF must contain real selectable text, not images of text.
- **Handwritten scores are not supported** for the same reason.
- **Complex rhythmic notation** (tuplets, syncopation) may cause beat-alignment errors.
- **Two-voice or three-voice systems** are partially supported — chords may be flagged as uncertain more often.

---

## Roadmap

- [x] Phase 1 — Web application (this repo)
- [ ] Phase 2 — Flutter mobile app consuming this backend
- [ ] Phase 3 — Flutter mobile app with 100% on-device processing (no server)

---

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

1. Fork the repo
2. Create your feature branch (`git checkout -b feature/my-feature`)
3. Commit your changes (`git commit -m 'Add my feature'`)
4. Push to the branch (`git push origin feature/my-feature`)
5. Open a Pull Request

---

## License

[MIT](LICENSE)
