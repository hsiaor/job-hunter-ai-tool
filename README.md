# Job Hunter AI Tool

## Installation and Instructions

The Job Hunter AI Tool is a CS 467 capstone web application that helps users
search for jobs and view ranked job matches. Users can enter a desired job
title, skills, location, and experience level, optionally upload a resume, and
review job results ranked by the backend ML scoring module.

## Hosted Project Access

Use this link for the easiest evaluation path:

[https://job-hunter-ai-tool.vercel.app/](https://job-hunter-ai-tool.vercel.app/)

Testing credentials: no account, login, or password is required. The app is
publicly accessible from a modern web browser.

Recommended browser: Chrome, Edge, Firefox, or Safari.

Note: the public hosted URL above is the frontend application. Backend URLs
shown later as `http://127.0.0.1:8000` are local-development URLs only. They
work after running the backend on your own machine and are not public hosted
API links.

## How to Examine and Use the Hosted App

1. Open [https://job-hunter-ai-tool.vercel.app/](https://job-hunter-ai-tool.vercel.app/).
2. If the Vercel app is not working or unresponsive, open: [https://job-hunter-ai-tool.onrender.com/](https://job-hunter-ai-tool.onrender.com/)
    When you see `{"status":"ok","message":"Job Hunting AI Web Tool backend is running"}`, return to the Vercel app.
3. In the search form, enter a target job title, such as `Software Engineer`.
4. Enter several skills, such as `Python`, `React`, `SQL`, or `Machine Learning`.
5. Enter a preferred location, such as `Remote`, `Portland`, or `Seattle`.
6. Select or enter an experience level if the field is shown.
7. Submit the search form.
8. Review the ranked job results. Each result may show the job title, company,
   location, score, and matched skills.
9. To test resume upload, use the resume upload area and choose a `.pdf` or
   `.docx` resume file. The app sends the file to the backend, extracts preview
   text, and can use resume content as additional search context.
10. Try another search with different skills or locations to compare ranking
   changes.

Example searches to try:

- Job title: `Software Engineer`; skills: `Python, SQL, Docker`; location:
  `Remote`
- Job title: `Data Analyst`; skills: `Python, Excel, Tableau`; location:
  `Seattle`
- Job title: `Machine Learning Engineer`; skills: `Python, NLP, scikit-learn`;
  location: `Remote`

## What the Project Demonstrates

- React frontend for job search, resume upload, and ranked results display.
- FastAPI backend endpoints for search, job listing, and resume upload.
- ML ranking module using normalized text, TF-IDF cosine similarity, and skill
  matching.
- MongoDB-backed job data used by the backend service.
- Automated tests for frontend components, backend contracts, and ranking logic.

## Repository Structure

```text
job-hunter-ai-tool/
  backend/      FastAPI app, API routes, schemas, seed data, resume parsing
  frontend/     React + Vite web app
  ml/           Job ranking module and ML-specific documentation
  pipeline/     Job-data collection, enrichment, and MongoDB helpers
  tests/        Backend, ML, integration, and contract tests
```

## Optional Local Installation

The hosted site above is the recommended way to evaluate the project. Local
installation is useful for development or code review. Running the full local
stack requires Python, Node.js, npm, and access to the backend job database or
equivalent local job data.

### Prerequisites

- Git
- Python 3.10 or newer
- Node.js 18 or newer
- npm
- MongoDB access for backend data-backed routes
- A terminal such as PowerShell, Command Prompt, Terminal, or Bash

Verify the main tools:

```bash
python --version
node --version
npm --version
```

### 1. Clone the Repository

```bash
git clone https://github.com/JobHunterAITool/job-hunter-ai-tool.git
cd job-hunter-ai-tool
```

If the repository is already provided as a ZIP file, extract it and open a
terminal in the extracted `job-hunter-ai-tool` directory.

### 2. Set Up the Python Environment

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r backend\requirements.txt
python -m pip install -r ml\requirements.txt
```

macOS or Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r backend/requirements.txt
python -m pip install -r ml/requirements.txt
```

Note: `backend/requirements.txt` includes the spaCy English model dependency.
If your environment does not install it automatically, run:

```bash
python -m spacy download en_core_web_sm
```

### 3. Configure Backend Environment Variables

Create a backend environment file if you are running the backend locally:

```text
backend/.env
```

Common values:

```text
MONGO_URI=mongodb+srv://JobSearchDBUser:HairyBeaver2026@cluster0.xmvpzab.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
MONGO_DB_NAME=job_hunter_ai
JOBS_COLLECTION_NAME=job_postings
CORS_ORIGINS=http://localhost:5173,http://127.0.0.1:5173,https://job-hunter-ai-tool.vercel.app
```

The backend expects job data to be available in MongoDB for `/search` and
`/jobs`. The root health check can run without exercising those data-backed
routes.

### 4. Run the Backend

From the repository root, with the Python environment activated:

```bash
uvicorn backend.main:app --reload --host 127.0.0.1 --port 8000
```

Check the local backend URL:

- Local backend health check: [http://127.0.0.1:8000/](http://127.0.0.1:8000/)

Expected health response:

```json
{
  "status": "ok",
  "message": "Job Hunting AI Web Tool backend is running"
}
```

### 5. Set Up and Run the Frontend

Open a second terminal in the repository root:

```bash
cd frontend
npm install
npm run dev
```

By default, the local frontend calls `http://127.0.0.1:8000`. To point it
somewhere else, create `frontend/.env`:

```text
VITE_API_BASE_URL=http://127.0.0.1:8000
```

Open the local frontend URL printed by Vite, usually:

[http://localhost:5173/](http://localhost:5173/)

### 6. Local Usage Checklist

1. Confirm the backend is running at `http://127.0.0.1:8000/`.
2. Confirm the frontend is running at `http://localhost:5173/`.
3. Enter a job title, skills, location, and experience level.
4. Submit the search.
5. Confirm ranked results appear.
6. Upload a `.pdf` or `.docx` resume and confirm the preview/upload response.

## Running Tests

Run backend and ML tests from the repository root:

```bash
python -m pytest
```

Run focused ranking tests:

```bash
python -m pytest tests/test_rank_jobs.py tests/test_rank_jobs_seed.py
```

Run frontend tests:

```bash
cd frontend
npm install
npm run test -- --run
```

If a full backend test run cannot find local MongoDB or generated job-data
fixtures, use the hosted site for evaluation and run the focused ML/frontend
tests that do not require the local database.

## Build Commands

Frontend production build:

```bash
cd frontend
npm install
npm run build
```

Backend production start command:

```bash
uvicorn backend.main:app --host 0.0.0.0 --port $PORT
```

## Main API Endpoints

These paths are served by the FastAPI backend. When running locally, use
`http://127.0.0.1:8000` as the base URL.

- `GET /` - backend health check
- `POST /search` - submit job search criteria and receive ranked jobs
- `GET /jobs?page=1&limit=20` - list available jobs
- `POST /upload-resume` - upload a PDF or DOCX resume for text extraction

Local endpoint examples:

```text
GET  http://127.0.0.1:8000/
GET  http://127.0.0.1:8000/docs
POST http://127.0.0.1:8000/search
GET  http://127.0.0.1:8000/jobs?page=1&limit=20
POST http://127.0.0.1:8000/upload-resume
```

The backend health check is `GET /`. There is no separate `/health` endpoint in
the current implementation.

Example `POST /search` payload:

```json
{
  "job_title": "Software Engineer",
  "skills": ["Python", "SQL", "Docker"],
  "location": "Remote",
  "experience_level": 1
}
```

## Data Pipeline
The `pipeline/` directory contains the data ingestion and enrichment scripts used to populate the MongoDB job dataset. The pipeline pulls job records from the Adzuna API, normalizes the returned JSON fields, follows available redirect URLs, and attempts to scrape the full external job posting page to collect a more complete job description.

After the raw job data is collected, the enrichment step adds derived fields used by the backend search and ranking flow. This includes extracting years of experience from the job description text, cleaning location values, separating city and state fields where possible, and preserving metadata for blocked, failed, or incomplete scrape attempts.

Pipeline output is stored in MongoDB Atlas in the `job_hunter_ai` database and the `job_postings` collection. The upload scripts connect through `pymongo` using the MongoDB Atlas connection string, verify the connection with a ping, select the target database and collection, and then insert or replace job records depending on the script being run. The FastAPI backend reads from this same MongoDB collection for the `/search` and `/jobs` endpoints.

MongoDB Atlas access requires both valid database credentials and network-level access. Even with the correct connection string, the connection will fail unless the client machine’s public IP address is added to the Atlas Network Access allowlist. This is required for local development, database updates, and any pipeline script that reads from or writes to the hosted database.

The crawler also includes reliability handling for rate limiting and blocked requests. Instead of relying on high concurrency, the current scraper uses slower request pacing, tracks blocked URLs, writes enriched records incrementally, and saves checkpoint progress so failed runs can resume without reprocessing the full dataset.

Additional helper scripts are included for pulling records from MongoDB, applying enrichment updates to existing records, replacing updated records back into the collection, deleting local generated files, and saving local backups before database writes.

## Troubleshooting

- If the hosted site does not load, refresh the page or try another browser.
- If local frontend searches fail, confirm the backend is running and
  `VITE_API_BASE_URL` points to the backend URL.
- If local backend startup fails on MongoDB or job-data setup, verify your
  `backend/.env` values and database access.
- If PowerShell blocks virtual environment activation, run:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

## Evaluation Recommendation

For grading or demonstration, start with the hosted web app:

[https://job-hunter-ai-tool.vercel.app/](https://job-hunter-ai-tool.vercel.app/)

No testing credentials are required. Use the example searches above to examine
the core search, ranking, and resume-upload workflows.
