---
type: concept
title: "Vibecoding Web App Workflow"
created: 2026-05-09
updated: 2026-05-09
tags:
  - vibecoding
  - web-development
  - fastapi
  - react
  - deployment
  - google-stitch
  - mcp
status: developing
related:
  - "[[LLM Wiki Pattern]]"
---

# Vibecoding Web App Workflow

Vibecoding is building software by describing what you want in plain language and letting AI write the code. You focus on ideas and design — the AI handles technical implementation. The term was coined by Andrej Karpathy (OpenAI co-founder) in 2025.

The workflow below covers the full process from idea to deployed website, using the stack: Claude + Google Stitch + FastAPI + React + Railway/Vercel.

---

## The Full Workflow

```
Your Idea
    ↓
user_flow.md  (write every click and screen)
    ↓
Claude  (turn flow into a design prompt)
    ↓
Google Stitch  (turn prompt into UI mockups)
    ↓
design.md via MCP  (give Claude the design spec)
    ↓
FastAPI (Python)    ←→    React (JavaScript)
   backend                   frontend
 localhost:8000           localhost:5173
    ↓                         ↓
         Live Server (preview locally)
                 ↓
         Push to GitHub
                 ↓
    Railway / Vercel (deploy)
                 ↓
       https://yourapp.com
```

---

## Step 1: Write a User Flow (`user_flow.md`)

Before touching any tool, write down every click and screen transition in your app as a numbered list.

**Why:** If you don't know where each button leads, the AI won't either. This is the brain of your app.

**Format:**
```
1. User opens app → sees homepage
2. User clicks "Sign Up" → sees registration form
3. User fills email + password → clicks Submit
4. If valid → goes to Dashboard
5. If invalid → sees error "Email already taken"
6. On Dashboard → sees list of tasks
7. User clicks "Add Task" → small form appears
8. User types task name → clicks Save → task appears in list
```

Rules: one numbered line per action, include what happens when things go wrong (error states), include every decision point.

---

## Step 2: Generate a Design Prompt with Claude

Paste your user flow into Claude and ask it to write a detailed UI design description as markdown. Claude describes colors, layouts, button positions, typography, and spacing for each screen.

This markdown becomes the instruction manual you feed into Google Stitch.

---

## Step 3: Generate UI Design with Google Stitch

**Google Stitch** (`stitch.withgoogle.com`) is a free Google Labs tool that turns text descriptions into visual UI mockups, powered by Gemini 2.5 Pro.

- Paste your Claude-generated design prompt into Stitch
- Stitch generates visual UI designs for each screen
- Refine by voice: "make buttons larger", "switch to dark mode"
- Can also upload hand-drawn sketches — Stitch digitizes them
- Export as a file or to Figma

**Free tier:** 350 generations/month.

---

## Step 4: Send Design to Claude via MCP (`design.md`)

**MCP (Model Context Protocol)** is a standard that lets Claude connect directly to your files and tools — like a USB port for AI.

Instead of copy-pasting the Stitch design into chat, MCP lets Claude read your `design.md` file directly. Claude then uses that design spec as reference when writing your frontend and backend code — it "sees" your design and generates code that matches it.

---

## Step 5: Build Backend with Python + FastAPI

**FastAPI** is a Python framework for building the backend (logic, data storage, API endpoints). Think of it as the kitchen: it prepares data and sends it to the frontend.

Key concepts:
- FastAPI creates **API endpoints** — URLs the frontend calls to get or send data
- Example: `GET /tasks` returns a list of tasks. `POST /tasks` creates a new one
- FastAPI auto-generates interactive docs at `http://localhost:8000/docs`

**To run locally:**
```bash
pip install fastapi uvicorn
uvicorn main:app --reload
# Server runs at http://localhost:8000
```

**Simple example:**
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/tasks")
def get_tasks():
    return [{"id": 1, "name": "Buy groceries"}]
```

---

## Step 6: Build Frontend with React

**React** is a JavaScript library for building everything the user sees and clicks on in the browser. Think of it as the dining room: it displays data and responds to user actions.

Key concepts:
- Built from **components** (reusable blocks: buttons, forms, cards)
- Fetches data from FastAPI backend via API calls
- Updates the page without full reloads (fast and smooth)
- Uses **JSX** — HTML-like syntax inside JavaScript

**To create and run:**
```bash
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev
# Frontend runs at http://localhost:5173
```

---

## Step 7: Preview Locally with Live Server

**Live Server** is a VS Code extension that hosts your files locally and auto-refreshes the browser every time you save — no manual F5 needed.

- Install from VS Code Extensions panel (search "Live Server" by Ritwick Dey)
- Right-click your HTML file → "Open with Live Server", or click "Go Live" in the status bar
- Runs at `http://localhost:5500`
- Keyboard shortcut: `Alt+L, Alt+O` to start / `Alt+L, Alt+C` to stop

Used for checking the frontend quickly during development. Not for production.

---

## Step 8: Deploy to the Internet

Deployment moves your app from your laptop to a cloud server so anyone can access it via a URL.

You deploy two parts separately:

| Part | Platform | URL |
|---|---|---|
| Frontend (React) | Vercel | `yourapp.vercel.app` |
| Backend (FastAPI) | Railway | `yourapi.railway.app` |

**Simplest path (Railway):**
1. Push code to GitHub: `git add . && git commit -m "ready" && git push`
2. Sign up at `railway.app` → connect GitHub
3. Create two services: one for FastAPI, one for React
4. Set environment variables (API URL, secrets)
5. Click Deploy → Railway builds and hosts both
6. Get your live URL

**Vercel** is also popular for React frontends: free, fast CDN, auto-deploys on every GitHub push.

---

## Required Software

| Tool | Purpose |
|---|---|
| Python 3.10+ | FastAPI backend |
| Node.js (LTS) | React frontend |
| VS Code | Code editor |
| Git | Track and push code |
| GitHub account | Store code, connect to Railway/Vercel |

**VS Code Extensions:**
- Live Server (by Ritwick Dey)
- Python (by Microsoft)
- ES7+ React snippets (by dsznajder)

---

## Frontend vs Backend Mental Model

| | Frontend (React) | Backend (FastAPI) |
|---|---|---|
| **What users see** | Yes | No |
| **Language** | JavaScript | Python |
| **Runs on** | User's browser | Server |
| **Handles** | UI, clicks, display | Data, logic, rules |
| **Local port** | 5173 | 8000 |
| **Deploy to** | Vercel | Railway |
