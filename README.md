# ai-resume-builder

AI Resume Builder is a web and CLI tool that helps users create professional, ATS-friendly resumes using AI. It generates resume content and templates from user-provided data (work history, skills, education) and offers customization, export (PDF), and integration options (OpenAI, user prompts, templates).

[Optional badges]
- Build status: ![build](https://img.shields.io/badge/build-passing-brightgreen)
- License: ![license](https://img.shields.io/badge/license-MIT-blue)
- OpenAI usage: ![openai](https://img.shields.io/badge/OpenAI-API-yellow)

Table of contents
- [Features](#features)
- [Demo](#demo)
- [Tech stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Local](#local)
  - [Docker](#docker)
- [Configuration](#configuration)
- [Usage](#usage)
  - [Web UI](#web-ui)
  - [CLI](#cli)
  - [API examples](#api-examples)
- [Templates & Customization](#templates--customization)
- [Deployment](#deployment)
  - [Vercel / Netlify](#vercel--netlify)
  - [Docker / Docker Compose](#docker--docker-compose)
- [Testing](#testing)
- [CI / CD](#ci--cd)
- [Security & Privacy](#security--privacy)
- [Contributing](#contributing)
- [Code of Conduct](#code-of-conduct)
- [License](#license)
- [Maintainers / Contact](#maintainers--contact)
- [Acknowledgements](#acknowledgements)
- [FAQ / Troubleshooting](#faq--troubleshooting)

Features
- Generate tailored resumes from structured user input using configurable AI prompts.
- Multiple resume templates (simple, modern, ATS-optimized).
- Export to PDF and Markdown.
- Web UI and CLI.
- Save / load user profiles and multiple resume versions (optional DB).
- Integration with OpenAI (or other LLM providers) with configurable prompt templates and model settings.
- Privacy-first design: user data kept local or encrypted server-side (configurable).

Demo
- Live demo URL (if deployed): https://your-deployment.example.com
- Screenshots: docs/screenshots/*.png

Tech stack
- Frontend: React (Next.js recommended) or Vue
- Backend: Node.js + Express / Fastify (or Next API routes)
- AI: OpenAI API (GPT models) — pluggable to others
- Storage: PostgreSQL / SQLite / MongoDB (optional)
- PDF generation: Puppeteer / Playwright or headless Chromium
- CI: GitHub Actions
- Container: Docker

Prerequisites
- Node.js >= 18
- npm or yarn
- (Optional) Docker & Docker Compose
- OpenAI API key (or equivalent LLM provider key)
- (Optional) Database (Postgres, SQLite file, or MongoDB)

Installation

Local
1. Clone the repo:
   git clone https://github.com/<owner>/ai-resume-builder.git
   cd ai-resume-builder
2. Install:
   npm install
   # or
   yarn install
3. Create .env (see Configuration)
4. Run:
   npm run dev
   # or
   yarn dev
5. Visit http://localhost:3000

Docker
1. Build:
   docker build -t ai-resume-builder .
2. Run:
   docker run -e OPENAI_API_KEY=your_key -p 3000:3000 ai-resume-builder

Configuration

Create a .env file in the project root with:

```env
# .env.example
NODE_ENV=development
PORT=3000

# OpenAI (or other LLM) credentials
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxxxxxx
OPENAI_MODEL=gpt-4o-mini  # choose default model

# Database (optional)
DATABASE_URL=postgres://user:password@localhost:5432/ai_resume_builder

# App settings
JWT_SECRET=replace-with-secure-random
SESSION_COOKIE_NAME=ai_resume_session
RESUME_DEFAULT_TEMPLATE=modern
MAX_TOKENS=2048

# Optional: SMTP or file storage settings for exports or notifications
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=...
SMTP_PASS=...
```

Environment variables
- OPENAI_API_KEY: required for AI generation.
- OPENAI_MODEL: model to call (e.g., gpt-4o-mini, gpt-4, gpt-3.5-turbo).
- DATABASE_URL: optional DB for saving resumes and profiles.
- JWT_SECRET: required if you use authentication.
- PORT: server port.

Usage

Web UI
- Start the app and open http://localhost:3000.
- Fill in your profile (contact, experience, education, skills).
- Select a template and tone (e.g., “concise professional”).
- Click "Generate" to produce the resume content using the AI assistant.
- Edit generated text inline and export as PDF.

CLI
- Example CLI command (if included in the repo):
  node cli/generate.js --profile ./examples/jane.json --template modern --output jane.pdf
- Flags:
  --profile: path to JSON input with user data
  --template: template name
  --output: output file path (.pdf or .md)
  --model: override default model

Input JSON format (example)
```json
{
  "name": "Jane Developer",
  "title": "Senior Software Engineer",
  "contact": {
    "email": "jane@example.com",
    "phone": "555-0100",
    "location": "San Francisco, CA",
    "linkedin": "https://linkedin.com/in/jane"
  },
  "summary": "Experienced software engineer...",
  "experience": [
    {
      "company": "Acme Corp",
      "position": "Senior Engineer",
      "startDate": "2020-06",
      "endDate": "Present",
      "summary": "Led a team..."
    }
  ],
  "education": [...],
  "skills": ["JavaScript", "Node.js", "React"]
}
```

API examples

POST /api/generate
- Request:
  POST /api/generate
  Content-Type: application/json
  Authorization: Bearer <JWT or API key>
  Body:
  {
    "profile": { ... },
    "template": "ats",
    "tone": "concise"
  }
- Response:
  {
    "resume": { "markdown": "...", "html": "...", "metadata": {...} },
    "pdfUrl": "/exports/jane-2026-05-14.pdf"
  }

cURL example:
```bash
curl -X POST "http://localhost:3000/api/generate" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $API_KEY" \
  -d '{"profile": {...}, "template":"modern"}'
```

Templates & Customization
- Templates are located in /templates (HTML + CSS).
- To add a template: create a new folder with template.html, template.css, and metadata.json.
- Prompt templates: prompts/ contains human-readable prompt templates used for generating summary, bullet points, and skill sections. Edit prompts to change tone, verbosity, or output formatting.

Prompt engineering tips
- Provide the AI with structured data, constraints (length, tone), and example outputs.
- Use system + user messages if using ChatCompletion-style APIs:
  - System: "You are an expert resume writer focused on ATS optimization."
  - User: Provide profile JSON and desired format.

Deployment

Vercel / Netlify (Next.js)
- Push to GitHub and connect the repo in the Vercel dashboard.
- Add environment variables (OPENAI_API_KEY, DATABASE_URL) in the project settings.
- Deploy automatically on push.

Docker Compose (example)
```yaml
version: "3.8"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - DATABASE_URL=${DATABASE_URL}
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: ai_resume_builder
    volumes:
      - db-data:/var/lib/postgresql/data
volumes:
  db-data:
```

Testing
- Unit tests: jest / vitest runner
- Integration tests: Playwright / Cypress for UI flows (generate, edit, export)
- Sample commands:
  npm run test
  npm run test:e2e

CI / CD
- Example GitHub Actions workflow:
  - Run lint and tests on push/PR
  - Build and publish Docker image on release
  - Deploy to Vercel on push to main

Security & Privacy
- Do not log sensitive PII in server logs.
- Use HTTPS in production.
- Encryption-at-rest for saved resumes (if storing).
- Give users the option to keep data local (no server-side storage).
- Follow provider policies (OpenAI data usage and retention settings). If needed, set "do not store" flags per provider.

Costs & Rate Limits
- Calling large LLMs can be costly — prefer a cost-optimized model for drafts and only use higher-cost models for final polishing.
- Implement request throttling and quotas.
- Show users estimated cost preview (optional).

Contributing
- Contributions welcome! Please:
  1. Fork the repo
  2. Create a feature branch: git checkout -b feat/my-feature
  3. Run tests and linters
  4. Open a PR with a clear description
- Follow contributor guidelines in CONTRIBUTING.md (add one if not present).

Code of Conduct
- This project follows the Contributor Covenant Code of Conduct. Please see CODE_OF_CONDUCT.md for details.

License
- MIT License — see LICENSE file.

Maintainers / Contact
- Maintainer: Your Name <you@example.com>
- GitHub: https://github.com/<owner>/ai-resume-builder
- For security issues, contact: security@example.com

Acknowledgements
- Built using OpenAI, Puppeteer, React, and contributors from the open-source community.

FAQ / Troubleshooting
- Q: Why is the AI generating irrelevant content?
  A: Provide more structured inputs and add examples in the prompt templates. Lower temperature for more deterministic output.
- Q: Generated PDF is misformatted?
  A: Check template CSS and headless browser version (Chromium). Use Puppeteer’s emulation of print styles.
- Q: How do I reduce cost?
  A: Use smaller models for initial drafts and only call larger models for final refinements. Cache generated sections.

Changelog
- Keep a CHANGELOG.md with notable releases.

Notes for maintainers
- Make prompt templates editable via the admin panel.
- Consider adding local AI model support (Llama.cpp / ONNX) for privacy-conscious deployments.
- Add analytics for feature usage (respecting privacy).
