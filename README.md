# DocScan-Vision

A command-line **computer vision** project that turns a casual photo of a document into a
clean, flat scan and tells you whether the capture quality is good enough.

## Overview
The pipeline applies classical computer-vision techniques taught in a Computer Vision
course (filtering, edge detection, contour analysis, homography, histogram equalisation,
adaptive thresholding) and combines them with a small machine-learning classifier that
judges image quality from interpretable image features.

```
photo -> edges + contours -> 4 corners -> perspective warp -> quality check -> enhance -> scan + JSON report
```

![pipeline example](docs/screenshots/sample_01_good_pipeline.jpg)
*Left to right: detected page boundary, rectified page, black-and-white scan.*

## Features
- **Module 1 - Document detection & rectification** (`detection.py`, `transform.py`)
- **Module 2 - Image enhancement** (`enhance.py`): non-local-means denoising, CLAHE, adaptive threshold
- **Module 3 - Quality analysis & classification** (`quality.py`, `classifier.py`): sharpness,
  brightness, contrast, edge density, skew; Random Forest vs. rule-based baseline
- Synthetic data generator with exact ground truth (no dataset download needed)
- CLI with `demo`, `train`, `scan`, `analyze` commands; batch mode; logging; JSON reports
- 17 automated tests (pytest)

## Technologies
Python 3.9+, OpenCV (headless), NumPy, scikit-learn, joblib, pytest. No GUI and no internet
access are required to run it.

## Project Structure
```
docscan/            source package (config, io_utils, preprocess, detection, transform,
                    enhance, quality, synthetic, classifier, pipeline, cli)
tests/              unit + end-to-end tests
docs/diagrams/      architecture, workflow, use-case, class, sequence diagrams
docs/screenshots/   example outputs
docs/results/       saved evaluation metrics
sample_data/        generated example photos (recreate with `demo`)
statement.md        problem statement
```

## Installation and Setup (step by step)
1. Install Python 3.9 or newer and Git.
2. Clone the repository and enter it:
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   ```
3. Create and activate a virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate          # Windows: .venv\Scripts\activate
   ```
4. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Running the Project
Run everything from the repository root.

```bash
# 1. Generate example document photos into sample_data/
python -m docscan demo --out sample_data --count 8

# 2. Train the quality classifier (about 10-20 seconds); saves models/quality_model.joblib
python -m docscan train --samples 80

# 3. Scan a whole folder (or pass a single image path to --input)
python -m docscan scan --input sample_data --output output

# 4. Print raw quality features for one image
python -m docscan analyze --input sample_data/sample_01_good.jpg
```
You can scan your own photos (`.jpg`, `.png`, `.bmp`, `.tif`) by pointing `--input` at them.
Without a trained model, `scan` still works and uses the rule-based quality check
(add `--no-model` to force it). Use `-v` for verbose logs; logs are also written to `docscan.log`.

### Output files (per input image `name`)
| File | Content |
|---|---|
| `name_1_detected.jpg` | original with detected page outline |
| `name_2_rectified.png` | perspective-corrected page |
| `name_3_enhanced.png` | denoised + contrast-enhanced colour page |
| `name_4_scan_bw.png` | black-and-white scan |
| `name_report.json` | corners, quality label + probabilities, features, skew, suggestions |

## Testing
```bash
python -m pytest -q
```
Tests cover corner ordering, detection accuracy against ground truth, fallback behaviour,
perspective output, feature behaviour under blur/darkness/low contrast, skew estimation,
model training, error handling for bad files, and a full CLI run.

## Results (synthetic hold-out set, 25 % of 320 pages)
| Method | Accuracy |
|---|---|
| Random Forest (4 features) | 100 % (5-fold CV 100 %) |
| Hand-written rule baseline | 86 % |

Synthetic data is easier than real photographs, so treat these numbers as a controlled
benchmark rather than a real-world guarantee. See the report for limitations.

## Troubleshooting
- `No trained model ... using rule-based quality check`: run the `train` command first.
- `Unsupported file type` / `Could not decode image`: use a valid image file.
- `Page boundary not detected`: photograph the page fully in frame on a contrasting surface.

## License
For academic use.
