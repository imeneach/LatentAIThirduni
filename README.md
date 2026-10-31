# LatentAI (Edit Detector)

**Shazam for photo edits.** Upload an edited photo and LatentAI shows you the Lightroom-style slider settings behind it, then lets you download them as a preset or LUT.

**Live demo:** https://latentaithirduni.onrender.com
*(The free server sleeps when idle. If the page is slow, wait about a minute and refresh.)*

---
---

## What it is

When a photo is exported, the editing settings are usually lost. If you love someone's look, you end up guessing slider by slider.

LatentAI works backwards from the result:

| You provide | What you get |
|---|---|
| An edited photo that still has Lightroom/Camera Raw metadata | The **exact** settings, read from the file |
| An edited photo **and** the original | **Fitted** slider values, each with an uncertainty, plus a confidence score |
| An edited photo only, no metadata | A clearly labeled **rough estimate** |

Results include tone curves and heat maps (brightness, saturation, clipping, warmth, change from original). You can download a Lightroom preset (`.xmp`), a 3D LUT (`.cube`), or the values as JSON, CSV or text.

## How to use the website

1. Open the site and click **Analyze a photo**or **deep analyses**
2. Choose your **edited photo**. Add the **original** too if you have it, because this is much more accurate.
3. Click **Analyze** and read the summary, confidence and parameters.
4. Open the **Curves**, **Heat maps** and **Download** tabs.
5. For a faster, shorter result, use **Analyze** in the top menu (quick analysis).

Your photos are processed in memory and are never stored.

## Run it locally

You need **Python 3.10 or newer**. `exiftool` is optional; the app falls back to a built-in reader without it.

```bash
# 1. Get the code
git clone https://github.com/imeneach/LatentAIThirduni.git
cd LatentAIThirduni

# 2. Create and activate a virtual environment
python -m venv venv
venv\Scripts\activate            # Windows
# source venv/bin/activate       # macOS / Linux

# 3. Install
pip install -r requirements.txt

# 4. Start the server
uvicorn app.main:app --reload
```

Then open **http://localhost:8000/**. Interactive API docs are at http://localhost:8000/docs.

**With Docker instead:**

```bash
docker build -t latentai .
docker run -p 8000:8000 latentai
```

**Run the tests:**

```bash
pip install -r requirements-dev.txt
pytest -q
```

**Settings (all optional):** `MAX_UPLOAD_MB` (default 25), `DATA_DIR` (default `./data`), `PORT` (default 8000, used in Docker).

## Project layout

```
app/
  main.py        API server and the /analyze pipeline
  xmp.py         reads exact Lightroom/Camera Raw metadata
  align.py       aligns the original to the edit (handles crop and resize)
  model.py       slider model and the optimiser that fits the settings
  fit.py         runs the fit and produces curves and confidence
  estimate.py    rough estimate when no original exists
  export.py      .xmp preset and .cube LUT writers
  static/        the website (index.html, analyze.html, images)
scripts/         benchmark and calibration tools
tests/           automated tests
Dockerfile       for deployment
```

## Who built what

| Person | Role and parts |
|---|---|
| **Imene Achour** ([@imeneach](https://github.com/imeneach)) | Project lead. Product idea and direction, website design and content, testing, deployment, and the project's submission materials. |
| **Claude (Anthropic's AI assistant)** | AI-assisted development under Imene's direction: help writing and debugging the Python pipeline and the slider-fitting model, the tests and benchmark.

AI tools were used heavily on this project, and every part was reviewed and tested by the team.

## Known limits

- Slider values come from our own model of Lightroom's sliders, so they are **close approximations**, not Lightroom's exact numbers, until calibrated against real exports.
- Temperature and tint are relative shifts, not Kelvin.
- Local brushes, masks and AI edits can't be explained by global sliders. The result warns you and lowers its confidence.
- Heavy cropping lowers accuracy.
- The free demo server is slow and sleeps when idle.

## What's next

Calibrate against real Lightroom exports, handle heavily cropped originals, add Clarity/Texture/Dehaze/HSL sliders, and train a model for the case with no original.

## Security and secrets

This repository contains **no API keys, passwords or private files**, and the app needs none to run. `.env`, `data/`, `venv/` and other local files are listed in `.gitignore`. If you add credentials later, keep them in environment variables and never commit them.

## License
Imene Achour all rights reserved
