# DictaHyperCube — Tesseract OCR for Rashi script

A Tesseract 5 LSTM model for printed Hebrew with a focus on Rashi script, fine-tuned from the official
`heb` model on 1.34M manually corrected text lines from 118 rabbinic books. Released with its test set.

| Model (line images, 1,120 lines) | CER | WER |
|---|---|---|
| **DictaHyperCube** | **0.79%** | **3.02%** |
| Best other public Hebrew Tesseract model (Boro `heb_rashi7`) | 2.99% | 11.87% |

CER/WER ignore punctuation and niqqud (see [Evaluation](#evaluation)). Details: the technical review (TODO link).

## Contents

| Path | What |
|---|---|
| `model/DictaHyperCube.traineddata` | the model (float, final checkpoint) |
| `test-set/images/*.png` | 14 scanned test pages, 300 dpi grayscale |
| `test-set/lines.json` | one entry per ground-truth line: bounding box + transcription |
| `ocr_example.py` | run the model from Python (pytesseract) |
| `evaluate.py` | reproduce the CER/WER above |
| `SHA256SUMS` | checksums of the model and test files |

## Usage

Install [Tesseract 5](https://tesseract-ocr.github.io/tessdoc/Installation.html). Command line:

```bash
tesseract page.png out --tessdata-dir model -l DictaHyperCube
```

Or copy `model/DictaHyperCube.traineddata` into your `tessdata` folder and use `-l DictaHyperCube`.

Python ([pytesseract](https://pypi.org/project/pytesseract/)):

```bash
pip install pytesseract pillow
python ocr_example.py page.png          # full page
python ocr_example.py line.png --line   # one cropped text line
python ocr_example.py page.png --tsv    # words with boxes and confidences
```

The core of it:

```python
import os, pytesseract
from PIL import Image
os.environ["TESSDATA_PREFIX"] = "model"   # folder holding DictaHyperCube.traineddata
print(pytesseract.image_to_string(Image.open("page.png"), lang="DictaHyperCube"))
```

## Test set

14 pages from various Rashi-script books, scanned at 300 dpi; none of their text occurs in the training data.
`test-set/lines.json` is a flat list:

```json
[
{"image": "001_p1.png", "line": 1, "bbox": [x, y, width, height], "text": "...", "body": true},
...
]
```

- `bbox` is in pixels of the page image, top-left origin.
- `line` is the line's index on its page (gaps are lines without a transcription).
- `body: false` marks running headers and page numbers (16 lines); the reported numbers use the 1,120 `body` lines.
- Transcriptions are diplomatic: abbreviation marks and punctuation as printed.

## Evaluation

```bash
pip install pytesseract pillow
python evaluate.py                       # DictaHyperCube, body lines, letters normalization
python evaluate.py --model other.traineddata --out hyp.jsonl
```

Each line is cropped exactly at its `bbox` and read with `--psm 13` (raw line). Normalization (`letters`):
Unicode NFC, niqqud and cantillation removed, every character other than Hebrew letters, digits and spaces
dropped. CER/WER are total edit distance over total reference length.

## License

- Model and test set (`model/`, `test-set/`: page images, bounding boxes, transcriptions):
  [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- Code (`*.py`): MIT.

See [LICENSE](LICENSE).

## Citation

```bibtex
@misc{shmidman2026dictahypercube,
      title={DictaHyperCube: A State-of-the-Art OCR Model For Rashi Script},
      author={Ometz Shmidman and Yonah TODO and Aryeh TODO and Shlomo TODO and Moshe Koppel and Joshua Geudalia and Avi Shmidman},
      year={2026},
      eprint={TODO},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```
