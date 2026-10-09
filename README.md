#  Comment Toxicity Detector

Real-time toxic comment detection for social media, powered by a fine-tuned **DistilBERT** model. A Chrome extension scans comments on the page, sends them to a local Flask API, and **blurs toxic ones** with a click-to-reveal warning.

##  About

This project detects harmful comments across six categories: `toxic`, `severe_toxic`, `obscene`, `threat`, `insult` and `identity_hate`. It has three parts: a DistilBERT model fine-tuned on the Jigsaw Toxic Comment dataset, a Flask API that serves predictions, and a Chrome extension (Manifest V3) that automatically blurs toxic comments on YouTube, Twitter/X, Instagram, Facebook and Reddit. A web dashboard (login/sign-up and overview pages) is also included.

##  Features

- Fine-tuned DistilBERT for multi-label toxicity classification
- Flask REST API (`/predict`) returning per-label probabilities and an `is_toxic` flag
- Chrome extension that scans comments and blurs toxic ones, with click-to-reveal
- Popup checker to test any text manually
- Web dashboard for the "LiveStream Moderation Engine" front end

##  Model Details

| | |
|---|---|
| **Base model** | `distilbert-base-uncased` |
| **Task** | Multi-label classification (6 labels) |
| **Dataset** | [Jigsaw Toxic Comment Classification](https://www.kaggle.com/c/jigsaw-toxic-comment-classification-challenge), 159,571 comments |
| **Split** | 90% train / 10% validation |
| **Max length** | 256 tokens |
| **Epochs / LR / Batch size** | 2 / 2e-5 / 8 |
| **Precision** | fp16 |
| **Final training loss** | ≈ 0.028 |
| **Threshold** | `is_toxic = true` if any label probability > 0.6 |

The trained weights (`model.safetensors`) and `train.csv` are not included in the repo because of their size. Download the dataset from Kaggle and re-run the training notebook, or place your own trained model in `backend/model/`.

##  Getting Started

**1. Clone and install**

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
pip install -r requirements.txt
```

**2. Add the trained model** to `backend/model/` (`config.json`, `model.safetensors`, `tokenizer.json`, `tokenizer_config.json`).

**3. Run the API**

```bash
cd backend
python app.py
```

The API runs at `http://127.0.0.1:5000`.

**4. Load the Chrome extension**

1. Open `chrome://extensions`
2. Enable **Developer mode**
3. Click **Load unpacked** and select the `ct-e` folder
4. Open YouTube, X, Reddit, etc. Toxic comments will be blurred automatically.

**5. Open the web UI (optional):** open `frontend/index.html` in your browser.

##  API

**`POST /predict`**

Request:
```json
{ "comment": "your text here" }
```

Response (values illustrative):
```json
{
  "toxic": 0.97,
  "severe_toxic": 0.04,
  "obscene": 0.61,
  "threat": 0.01,
  "insult": 0.92,
  "identity_hate": 0.03,
  "is_toxic": true
}
```

##  Limitations

- Trained on Wikipedia comments, so accuracy on slang, sarcasm or other platforms may vary.
- Per-label evaluation metrics (F1 / ROC-AUC) are not reported yet.
- Social media sites change their HTML often, so the selectors in `content.js` may need updating.
- The login/sign-up UI is front-end only (uses `localStorage`) and is not real authentication.
- Automated moderation can produce false positives and negatives, so use it as an aid, not a final decision-maker.

##  Roadmap

- [ ] Per-label metrics and threshold tuning
- [ ] Batch predictions to reduce API calls
- [ ] Extension settings (threshold, per-site toggle)
- [ ] Real backend authentication and a moderation stats dashboard
- [ ] Dockerize the API

##  Tech Stack

Python · PyTorch · Hugging Face Transformers · Flask · JavaScript (Chrome Extension MV3) · HTML/CSS

##  Acknowledgements

Thanks to Jigsaw / Conversation AI for the dataset and Hugging Face for Transformers and DistilBERT.
