# Comment-Toxicity-Detector
Real-time toxic comment detection for social media, powered by a fine-tuned DistilBERT model. A Chrome extension scans comments on the page, sends them to a local Flask API, and blurs toxic ones with a click-to-reveal warning.
Multi-label classification across 6 toxicity categories: toxic, severe_toxic, obscene, threat, insult, identity_hate.
 Features
Fine-tuned DistilBERT for multi-label toxicity classification (6 labels, sigmoid outputs)
Flask REST API (/predict) that returns per-label probabilities and an is_toxic flag
Chrome extension (Manifest V3) that auto-scans comments and blurs toxic ones, with click-to-reveal
Popup checker to manually test any text
Web dashboard (login/sign-up UI + dashboard) for the "LiveStream Moderation Engine" front end
Platform support: YouTube, Twitter/X, Instagram, Facebook, Reddit (via per-site CSS selectors)
 Architecture
┌────────────────────┐   POST /predict    ┌──────────────────────┐
│  Chrome Extension  │ ─────────────────► │   Flask API          │
│  (content.js scans │  {"comment": ...}  │   DistilBERT model   │
│   page comments)   │ ◄───────────────── │   (PyTorch)          │
└────────────────────┘   probs + is_toxic └──────────────────────┘
        │ blur toxic comments
        ▼
   Click to reveal / hide
 Project Structure
.
├── backend/
│   ├── app.py              # Flask API (loads the model, serves /predict)
│   └── model/              # Fine-tuned DistilBERT (config, tokenizer, model.safetensors)
├── ct-e/                   # Chrome extension
│   ├── manifest.json
│   ├── content.js          # Scans & blurs toxic comments
│   ├── popup.html / popup.js
│   └── background.js
├── frontend/               # Landing/login page + dashboard
│   ├── index.html
│   └── dashboard.html
├── training/
│   └── training_code_Colab.ipynb   # Model training notebook (Google Colab)
├── requirements.txt
└── README.md
 Model
	
Base model	distilbert-base-uncased
Task	Multi-label sequence classification (6 labels)
Dataset	Jigsaw Toxic Comment Classification Challenge (159,571 Wikipedia comments)
Split	90% train / 10% validation (random_state=42)
Max sequence length	256 tokens
Epochs / LR / Batch	2 / 2e-5 / 8
Precision	fp16
Training time	~50 min on a Colab GPU
Final training loss	≈ 0.028 (BCE)
Decision threshold	is_toxic = True if any label probability > 0.6

The model weights (model.safetensors) and train.csv are not stored in this repo (see .gitignore). Download the dataset from Kaggle and re-run the training notebook, or place your trained model in backend/model/.

 Getting Started
1. Clone the repo
bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
2. Install dependencies
bash
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
3. Add the trained model

Train it with the Colab notebook in training/, then download the saved toxicity_model/ folder and place its contents in backend/model/:

backend/model/
├── config.json
├── model.safetensors
├── tokenizer.json
└── tokenizer_config.json
4. Run the API
bash
cd backend
python app.py

The API runs at http://127.0.0.1:5000.

5. Load the Chrome extension
Open chrome://extensions
Enable Developer mode
Click Load unpacked and select the ct-e/ folder
Visit YouTube / X / Reddit etc. — toxic comments will be blurred automatically
6. Open the web UI (optional)

Open frontend/index.html in your browser (or serve it with python -m http.server).

 API Reference
POST /predict

Request

json
{ "comment": "You are an idiot" }

Response

json
{
  "toxic": 0.97,
  "severe_toxic": 0.04,
  "obscene": 0.61,
  "threat": 0.01,
  "insult": 0.92,
  "identity_hate": 0.03,
  "is_toxic": true
}

(Values shown are illustrative.)

 Training

The full training pipeline is in the Colab notebook: load the Jigsaw CSV → tokenize with DistilBertTokenizerFast → fine-tune DistilBertForSequenceClassification with Hugging Face Trainer using problem_type="multi_label_classification" → save to toxicity_model/.

 Limitations
Trained on Wikipedia talk-page comments, so performance on slang, sarcasm, code-mixed text, or other platforms may vary.
No validation metrics (F1 / ROC-AUC) are reported yet; adding per-label evaluation is a good next step.
Social media sites change their HTML often, so the CSS selectors in content.js may need updating.
The login/sign-up UI in frontend/ is front-end only (state is stored in localStorage) and is not real authentication.
Automated moderation can produce false positives/negatives. Use it as an aid, not a final decision-maker.
 Roadmap
 Report per-label precision / recall / F1 / ROC-AUC
 Per-label threshold tuning
 Batch predictions to reduce API calls
 Extension settings (threshold, enable/disable per site)
 Real backend authentication and a moderation stats dashboard
 Dockerize the API
 Tech Stack

Python · PyTorch · Hugging Face Transformers · Flask · JavaScript (Chrome Extension MV3) · HTML/CSS

 License

Add a license of your choice (e.g. MIT) in a LICENSE file.

 Acknowledgements
Jigsaw / Conversation AI for the dataset
Hugging Face for Transformers and DistilBERT
