# DataScoop Customer Survey Tool

## Overview
Build a self-contained, deployable survey application that sends a 2-question survey to ~50 users via email, collects individualized responses through unique URLs, and stores results in PostgreSQL.

## Tech Stack
- **Runtime**: Node.js AWS Lambda (behind API Gateway)
- **Database**: PostgreSQL (assume connection string via env var `DATABASE_URL`)
- **Email**: SendGrid API (assume API key via env var `SENDGRID_API_KEY`)
- **Frontend**: Static HTML/CSS/JS served by the Lambda (no framework — keep it lightweight)

## Database Schema
Create a migration script that sets up:
- `survey_recipients` table: user_id, email, name, unique_token (UUID), survey_sent_at, responded_at, response_q1 (boolean), response_q2 (boolean)
- Index on `unique_token` for fast lookups

## Core Features

### 1. Survey Page (GET /survey?token=<unique_token>)
- Validate the token exists and hasn't already been responded to
- If already responded, show a friendly "Thank you, you've already submitted" screen
- If valid, render a clean, elegant, one-question-at-a-time survey flow (Typeform-style):
  - **Q1**: "Would your workflow be impacted if you no longer had access to DataScoop?" → Yes / No buttons
  - **Q2**: "Do you want to be kept in the distribution of the weekly emails?" → Yes / No buttons
- After Q2, show a confirmation/thank-you screen
- Mobile-responsive. Smooth transitions between questions. Large click targets. Progress indicator.

### 2. Submit Endpoint (POST /survey/submit)
- Accepts: { token, q1: boolean, q2: boolean }
- Validates token, checks for duplicate submission, stores responses, sets responded_at timestamp
- Returns success/error JSON

### 3. Send Survey Emails (POST /admin/send — protect with a simple API key header `x-admin-key`)
- Accepts: { recipients: [{ user_id, email, name }] }
- For each recipient: generate a unique_token, insert into survey_recipients, send email via SendGrid
- Email should be clean and professional with a clear CTA button linking to the survey URL
- Return summary: { sent: number, failed: number }

### 4. Resend to Non-Respondents (POST /admin/resend — same admin key protection)
- Query survey_recipients WHERE responded_at IS NULL
- Resend the survey email to each using their existing unique_token
- Return summary: { resent: number }

### 5. Results Dashboard (GET /admin/results — same admin key protection)
- Simple HTML page showing:
  - Total sent / total responded / response rate
  - Breakdown of Yes/No for each question
  - Table of all recipients with their status (responded or pending)

## Design Direction
- Typeform-inspired: centered content, generous whitespace, large typography, one question per screen
- Subtle animations on transitions (CSS only, no heavy libraries)
- Color palette: clean neutrals with a single accent color for CTAs
- No branding logos needed — keep it minimal

## Deployment
- Provide a `serverless.yml` or SAM template for deployment
- Include a README with: setup instructions, env vars needed, how to run the migration, how to trigger the initial send, how to resend

## File Structure
Keep it simple — aim for a flat structure:
/
├── handler.js          (Lambda handler with all routes)
├── db.js               (PostgreSQL connection + queries)
├── email.js            (SendGrid integration)
├── templates/          (HTML templates for survey + emails)
├── migration.sql       (Database setup script)
├── serverless.yml      (Deployment config)
├── package.json
└── README.md