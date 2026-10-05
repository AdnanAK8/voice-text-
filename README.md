# Voice-to-Text Web Application using AWS

Speak into your browser, get text back. The project is split into two independent parts:

```
voice-to-text/
├── backend/     Flask API (Python) – talks to AWS S3 + Amazon Transcribe
│   ├── app.py
│   ├── requirements.txt
│   ├── .env.example
│   ├── .gitignore
│   └── iam-policy.json
├── frontend/    Static website (HTML/CSS/JS) – records audio, calls the API
│   ├── index.html
│   ├── config.js        <- set the backend address here
│   ├── css/style.css
│   └── js/script.js
└── docs/
    ├── architecture.md
    ├── aws-setup.md
    ├── testing.md
    └── viva-questions.md
```

## How it works
1. Browser records audio with `MediaRecorder` and sends it to `POST /api/transcribe`.
2. Backend saves the audio in a private S3 bucket and starts an Amazon Transcribe job.
3. Browser asks `GET /api/transcribe/<job_id>` every 2 seconds.
4. When finished, the backend returns the text and deletes the audio, transcript and job.

No AWS keys exist in the frontend or in the code.

## Run locally (after AWS setup, see docs/aws-setup.md Part A)

**1. Backend** (terminal 1)
```bash
cd backend
python -m venv venv
# Windows:  venv\Scripts\activate        Mac/Linux:  source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env        # Windows: copy .env.example .env   then edit .env
aws configure               # one time; uses an IAM user with the policy in iam-policy.json
python app.py
```
Open http://127.0.0.1:5000/api/health – you should see `"status": "ok"`.

**2. Frontend** (terminal 2)
```bash
cd frontend
python -m http.server 5500
```
Open http://localhost:5500 in Chrome or Edge, click **Start Recording**, speak, click **Stop Recording**.

The frontend must be served from `localhost` or `https` or the browser blocks the microphone.

## Common errors
| Problem | Fix |
|---|---|
| "Cannot reach the server" | Backend not running, or `config.js` has the wrong address |
| CORS error in browser console | Add your frontend address to `FRONTEND_ORIGINS` in `backend/.env`, restart backend |
| `NoCredentialsError` in backend log | Run `aws configure` (local) or attach an IAM Role (EC2) |
| "AWS service unavailable" | Check bucket name/region in `.env` and the IAM policy |
| "Server is not configured" | `S3_BUCKET_NAME` missing in `.env` |
| Microphone blocked | Click the lock icon in the address bar and allow microphone |

## Deleting everything afterwards
See the cleanup checklist at the end of `docs/aws-setup.md`.
