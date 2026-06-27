# 🤖 AI HR Recruitment Assistant

An end-to-end AI-powered recruitment automation system built with **n8n**, **Groq AI**, **Gmail**, and **Google Sheets**. Recruiters can submit resumes via a live web portal — the AI evaluates each candidate, logs results to Google Sheets, and sends automated emails.

![AI HR Recruitment Assistant](https://img.shields.io/badge/Built%20with-n8n-orange) ![Groq AI](https://img.shields.io/badge/AI-Groq%20llama--3.3--70b-blue) ![Google Sheets](https://img.shields.io/badge/Tracker-Google%20Sheets-green) ![GitHub Pages](https://img.shields.io/badge/Deployed-GitHub%20Pages-black)

---

## 🌐 Live Demo

**Recruiter Portal:** [https://mastans22602.github.io/Hr-Recruitment/](https://mastans22602.github.io/Hr-Recruitment/)

---

## 📋 What It Does

| Step | Action |
|------|--------|
| 1️⃣ **Trigger** | Resume received via web form (GitHub Pages UI) or Gmail email with PDF attachment |
| 2️⃣ **Extract** | PDF text is extracted from the resume attachment |
| 3️⃣ **AI Evaluate** | Groq AI (llama-3.3-70b) scores the candidate 1–10 and returns structured JSON |
| 4️⃣ **Decision** | Score > 7 → Shortlisted · Score 5–7 → Maybe · Score < 5 → Archived |
| 5️⃣ **Log** | Candidate data logged to the correct Google Sheets tab automatically |
| 6️⃣ **Notify** | Shortlisted candidates receive a confirmation email; hiring manager is notified |
| 7️⃣ **Display** | UI polls n8n every 5 seconds and displays the live ATS score + decision |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    RECRUITER PORTAL (GitHub Pages)          │
│           mastans22602.github.io/Hr-Recruitment             │
└────────────────────────┬────────────────────────────────────┘
                         │ POST (multipart/form-data + PDF)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                        n8n WORKFLOW 1                       │
│                  AI HR Recruitment Assistant                 │
│                                                             │
│  Webform Trigger ──┐                                        │
│                    ├──▶ Merge ──▶ Normalize Binary          │
│  Gmail Trigger ────┘             ──▶ Extract PDF Text       │
│    └──▶ Get Message                  ──▶ Basic LLM Chain    │
│                                          (Groq AI)          │
│                                      ──▶ Parse AI Response  │
│                                      ──▶ IF Shortlisted?    │
│                                          ├── ✅ Log Sheet   │
│                                          │   Email Candidate│
│                                          │   Notify Manager │
│                                          ├── 🤔 Maybe Sheet │
│                                          └── ❌ Archive Sheet│
└─────────────────────────────────────────────────────────────┘
                         │ GET ?email=...  (polling every 5s)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                        n8n WORKFLOW 2                       │
│                      HR Result Fetcher                      │
│                                                             │
│  GET Webhook ──▶ Extract Email ──▶ Read All 3 Sheets        │
│                                 ──▶ Find Row by Email       │
│                                 ──▶ Return JSON Result      │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
              UI updates with live score + decision
```

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **n8n** | Workflow automation (self-hosted via Docker) |
| **Groq AI** (llama-3.3-70b-versatile) | Resume evaluation and ATS scoring |
| **Gmail** | Resume intake via email + sending notifications |
| **Google Sheets** | Candidate tracker (Shortlisted / Maybe / Archived tabs) |
| **ngrok** | Public tunnel for n8n webhook access |
| **GitHub Pages** | Hosts the recruiter portal UI |
| **HTML / CSS / JS** | Frontend portal with polling for live results |

---

## 📁 Repository Structure

```
Hr-Recruitment/
│
├── index.html                          # Recruiter portal UI (GitHub Pages)
├── hr_recruitment_assistant_v2.json    # n8n Workflow 1 — Main pipeline
├── hr_result_fetcher.json              # n8n Workflow 2 — Result polling
└── README.md                           # This file
```

---

## ⚙️ Setup Guide

### Prerequisites
- n8n running via Docker (`docker-compose up -d`)
- ngrok with a static domain configured
- Google Cloud project with Gmail API + Google Sheets API enabled
- Groq API key (free at [console.groq.com](https://console.groq.com))

---

### 1. Google Sheets Setup

Create a new Google Sheet called **HR Recruitment Tracker** with 3 tabs:

**Tab names:** `Shortlisted` · `Maybe` · `Archived`

Add these headers in **Row 1** of each tab:
```
Name | Email | Phone | Current Role | Experience (Years) | ATS Score | Skills | Education | Summary | Status | Date
```

Copy the Sheet ID from the URL:
```
https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit
```

---

### 2. n8n Credentials Setup

In n8n → **Credentials**, create:

| Credential | Type |
|-----------|------|
| Gmail account | Gmail OAuth2 |
| Google Sheets account | Google Sheets OAuth2 |
| Groq account | Groq API (paste API key) |

---

### 3. Import Workflows into n8n

**Workflow 1 — Main Pipeline:**
- n8n → New Workflow → Import from file → `hr_recruitment_assistant_v2.json`
- Update `YOUR_GOOGLE_SHEET_ID` in all 3 Google Sheets nodes
- Update `hiring.manager@yourcompany.com` in Notify Hiring Manager node
- Connect Gmail, Google Sheets, and Groq credentials
- **Publish → Active**

**Workflow 2 — Result Fetcher:**
- n8n → New Workflow → Import from file → `hr_result_fetcher.json`
- Update `YOUR_GOOGLE_SHEET_ID` in all 3 Google Sheets nodes
- Connect Google Sheets credentials
- **Publish → Active**
- Copy the **Production URL** (e.g. `https://your-ngrok-domain.ngrok-free.dev/webhook/hr-result`)

---

### 4. Configure the Frontend

Open `index.html` and update the 3 config values at the top of the `<script>` section:

```js
const WEBHOOK_URL = 'https://your-ngrok-domain.ngrok-free.dev/webhook/hr-recruitment';
const POLL_URL    = 'https://your-ngrok-domain.ngrok-free.dev/webhook/hr-result';
const SHEET_URL   = 'https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit';
```

---

### 5. Deploy to GitHub Pages

```bash
git add .
git commit -m "Deploy AI HR Recruitment Assistant"
git push
```

Then in your GitHub repo → **Settings** → **Pages** → Source: `main` branch → `/ (root)` → **Save**

---

## 🎯 AI Scoring Logic

The AI (Groq llama-3.3-70b) evaluates each resume and returns:

```json
{
  "name": "Candidate Name",
  "email": "candidate@email.com",
  "phone": "+91 9876543210",
  "skills": ["Python", "SQL", "Machine Learning"],
  "experience_years": 5,
  "education": "B.Tech Computer Science",
  "current_role": "Data Analyst",
  "ats_score": 8,
  "summary": "Strong candidate with relevant experience...",
  "decision": "Shortlisted"
}
```

| ATS Score | Decision | Action |
|-----------|----------|--------|
| > 7 | ✅ Shortlisted | Logged to Shortlisted tab · Email sent to candidate · Hiring manager notified |
| 5 – 7 | 🤔 Maybe | Logged to Maybe tab |
| < 5 | ❌ Archived | Logged to Archived tab |

---

## 🔄 How Polling Works

1. Recruiter submits resume via the portal
2. UI immediately shows **"AI Screening..."** badge
3. n8n processes the resume in the background (~30 seconds)
4. UI polls `/webhook/hr-result?email=candidate@email.com` every **5 seconds**
5. Once the result appears in Google Sheets, the poll returns it
6. UI updates automatically with the real ATS score, decision, and candidate details
7. Max polling time: **90 seconds** (18 attempts × 5s)

---

## 📸 Screenshots

### Recruiter Portal
- Clean submission form with PDF upload
- Real-time "AI Screening..." badge while processing
- Live ATS score bar and Shortlisted/Maybe/Archived badge after evaluation

### Google Sheets Tracker
- Three tabs auto-populated by the AI
- Columns: Name, Email, Phone, Role, Experience, ATS Score, Skills, Education, Summary, Status, Date

---

## 🚀 Future Improvements

- [ ] Add WhatsApp notification for shortlisted candidates
- [ ] Add calendar scheduling integration for shortlisted candidates
- [ ] Build a dashboard to view all candidates across tabs
- [ ] Add multi-language resume support
- [ ] Role-specific scoring criteria (different weights per job)
- [ ] Admin login to protect the recruiter portal

---

## 👨‍💻 Author

**Mastan Shaik**
- GitHub: [@mastans22602](https://github.com/mastans22602)
- Portfolio: [mastans22602.github.io/Hr-Recruitment](https://mastans22602.github.io/Hr-Recruitment/)

---

## 📄 License

MIT License — free to use and modify.
