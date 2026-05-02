# FeedbackFusion — Survey & Feedback Platform

> A full-stack web application for creating, distributing, and analyzing surveys. Build forms with multiple question types, share them publicly or via private code, and view live response analytics — all in one place.

![Python](https://img.shields.io/badge/Python-Flask-3776AB?logo=python)
![MySQL](https://img.shields.io/badge/Database-MySQL-4479A1?logo=mysql)
![Bootstrap](https://img.shields.io/badge/UI-Bootstrap%205-7952B3?logo=bootstrap)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Database Schema](#database-schema)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Database Setup](#database-setup)
  - [Configuration](#configuration)
  - [Running the App](#running-the-app)
- [Application Pages & Routes](#application-pages--routes)
- [Route Reference](#route-reference)
- [Authentication & Session Management](#authentication--session-management)
- [Survey Lifecycle](#survey-lifecycle)
- [Question Types](#question-types)
- [Analytics Engine](#analytics-engine)
- [Helper Module](#helper-module)
- [Frontend Architecture](#frontend-architecture)
- [Known Issues & Security Considerations](#known-issues--security-considerations)
- [Future Improvements](#future-improvements)

---

## Overview

**FeedbackFusion** is a multi-user survey platform built with Flask and MySQL. Registered users can create surveys containing two types of questions — multiple choice and free text — set their visibility (public, link-only, or hidden), and share them using a numeric survey code. Respondents fill out surveys through a dynamic form and submit answers in a single JSON payload. Survey owners can then view per-question analytics: percentage breakdowns for multiple-choice questions and a full list of free-text responses.

---

## Features

| Feature | Description |
|---|---|
| **User Registration & Login** | Account creation with SHA-256 hashed passwords; session-based authentication |
| **Create Surveys** | Dynamic form builder supporting multiple-choice and free-text questions |
| **Visibility Control** | Three visibility modes: Public, Link-only, Hidden |
| **Survey Code Access** | Every survey gets a unique numeric ID used as a shareable access code |
| **Public Survey Discovery** | Browse and answer all active public surveys from one page |
| **Answer Surveys** | Respondents fill out rendered survey forms; one response per user per survey enforced |
| **Survey Status Toggle** | Owners can mark surveys as Completed or bring them back Online |
| **Delete Surveys** | Owners can permanently delete a survey and all its data via a stored procedure |
| **Response Analytics** | Per-question stats: percentage breakdown for options, full text for free-text answers |
| **Animated Homepage** | Typewriter greeting effect and Lottie animation on the landing page |
| **Responsive UI** | Bootstrap 5 navbar with mobile toggle; card-based survey grid |

---

## Tech Stack

### Backend

| Library | Version | Purpose |
|---|---|---|
| `Flask` | Latest | Web framework, routing, templating |
| `flask-session` | Latest | Server-side filesystem session storage |
| `flask-mysqldb` | Latest | MySQL database connector |
| `hashlib` | Built-in | SHA-256 password hashing |
| `json` | Built-in | JSON serialization for survey/answer payloads |
| `datetime` | Built-in | Timestamping survey creation and responses |

### Database

- **MySQL** — relational database storing users, surveys, questions, options, responses, and answers
- MySQL stored procedures (`InsertSurveyData`, `DeleteSurvey`) handle atomic multi-table operations

### Frontend

| Library | Source | Purpose |
|---|---|---|
| Bootstrap 5.3.3 | CDN (jsDelivr) | Responsive layout, cards, forms, buttons, navbar |
| DotLottie Player | CDN (unpkg) | Animated illustrations on home and empty-state pages |
| KUTE.js | CDN (jsDelivr) | Animation utility (included on homepage) |

> No JavaScript framework is used — all interactive UI (dynamic form builder, survey renderer, answer submission) is written in **vanilla JavaScript**.

---

## Project Structure

```
feedbackfusion-main/
├── app.py                        # Flask application — all routes and business logic
├── DataModel.mwb                 # MySQL Workbench data model file
├── LICENSE                       # MIT License
│
├── modules/
│   └── functions.py              # Shared helpers: authenticate(), mysql_db()
│
├── static/
│   ├── images/
│   │   ├── logo.png              # App logo (used in navbar and favicon)
│   │   └── user.png              # Default user avatar
│   └── Animation - 1717518109739 (1).json   # Lottie animation for homepage
│
├── templates/
│   ├── layout_base.html          # Base Jinja2 template (navbar + block structure)
│   ├── index.html                # Homepage — survey code entry + animated greeting
│   ├── signIn.html               # Login form
│   ├── signUp.html               # Sign-up redirect stub
│   ├── register.html             # Registration form
│   ├── create_survey.html        # Dynamic survey builder
│   ├── my_surveys.html           # Survey dashboard — owner view
│   ├── public_surveys.html       # Browse public surveys
│   ├── ans_survey.html           # Survey answering page
│   ├── analytics.html            # Per-survey response analytics
│   ├── about_us.html             # About page
│   └── errors.html               # Generic error/success message page
│
└── flask_session/                # Server-side session files (auto-generated, gitignore)
```

---

## Database Schema

The MySQL database is named `surveyfeedback`. The full visual schema is in `DataModel.mwb` (open with MySQL Workbench). Below is a summary of all tables and their roles.

### Tables

#### `users_table`
Stores registered user accounts.

| Column | Type | Notes |
|---|---|---|
| `user_id` | INT (PK, AUTO) | Primary key |
| `user_name` | VARCHAR | Must be alphanumeric, min 7 chars |
| `passcode` | VARCHAR | SHA-256 hex digest |
| `email_address` | VARCHAR | Stored as entered |

---

#### `surveys_table`
Stores each survey created by a user.

| Column | Type | Notes |
|---|---|---|
| `survey_id` | INT (PK, AUTO) | Numeric code shared with respondents |
| `user_id` | INT (FK) | Owner — references `users_table` |
| `survey_title` | VARCHAR | |
| `survey_description` | TEXT | |
| `brand_logo` | VARCHAR | (reserved for future use) |
| `status` | INT (FK) | References `status_table` |
| `visibility` | INT (FK) | References `visibility_table` |
| `date_created` | DATE | |

---

#### `status_table`
Lookup table for survey status values.

| `status_id` | `status_description` |
|---|---|
| 1 | Completed |
| 2 | Online |
| 3 | (reserved/draft) |

---

#### `visibility_table`
Lookup table for survey visibility modes.

| `visibility_id` | `visibility_description` |
|---|---|
| 1 | Public |
| 2 | Link |
| 3 | Hidden |

---

#### `questions`
Stores questions belonging to each survey.

| Column | Type | Notes |
|---|---|---|
| `QuestionId` | INT (PK, AUTO) | |
| `SurveyId` | INT (FK) | References `surveys_table` |
| `Description` | VARCHAR | Question text |
| `order` | INT | Display order within the survey |
| `questiontype_id` | INT | `1` = Options (multiple choice), `2` = Free Text |

---

#### `question_options`
Stores individual answer options for multiple-choice questions.

| Column | Type | Notes |
|---|---|---|
| `Id` | INT (PK, AUTO) | |
| `QuestionId` | INT (FK) | References `questions` |
| `OptDesc` | VARCHAR | Option text |
| `order` | INT | Display order within the question |

---

#### `response`
One record per user per survey submission.

| Column | Type | Notes |
|---|---|---|
| `response_id` | INT (PK, AUTO) | |
| `survey_id` | INT (FK) | |
| `user_id` | INT (FK) | |
| `date` | DATE | Submission date |

---

#### `answer`
One record per question answered in a response.

| Column | Type | Notes |
|---|---|---|
| `answer_id` | INT (PK, AUTO) | |
| `response_id` | INT (FK) | |
| `question_id` | INT (FK) | |
| `answer` | TEXT | Populated for free-text questions only |

---

#### `answer_option`
Links a multiple-choice answer to the selected option.

| Column | Type | Notes |
|---|---|---|
| `answer_option_id` | INT (PK, AUTO) | |
| `answer_id` | INT (FK) | References `answer` |
| `question_option_id` | INT (FK) | References `question_options` |

---

### Stored Procedures

#### `InsertSurveyData`
Called from `/create_survey`. Accepts the user ID, title, description, visibility, date, and a JSON string of all questions and options. Handles the full multi-table insert atomically.

```sql
CALL InsertSurveyData(user_id, title, desc, visibility, date, json_questions)
```

#### `DeleteSurvey`
Called from `/delete_survey`. Deletes a survey and all related records (questions, options, responses, answers) in the correct foreign-key order.

```sql
CALL DeleteSurvey(survey_id)
```

---

## Getting Started

### Prerequisites

- Python 3.8+
- MySQL 8.0+
- pip

### Installation

```bash
# 1. Unzip or clone the project
cd feedbackfusion-main

# 2. Create and activate a virtual environment (recommended)
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install Python dependencies
pip install flask flask-session flask-mysqldb
```

### Database Setup

1. Open MySQL Workbench and import `DataModel.mwb` to review the schema, or run the DDL manually.
2. Create the database and tables:

```sql
CREATE DATABASE surveyfeedback;
USE surveyfeedback;

-- Create tables: users_table, surveys_table, status_table, visibility_table,
-- questions, question_options, response, answer, answer_option
-- (use DataModel.mwb as reference for full DDL)

-- Seed lookup tables
INSERT INTO status_table (status_id, status_description) VALUES (1,'Completed'),(2,'Online'),(3,'Draft');
INSERT INTO visibility_table (visibility_id, visibility_description) VALUES (1,'Public'),(2,'Link'),(3,'Hidden');
```

3. Create the stored procedures `InsertSurveyData` and `DeleteSurvey` (DDL can be extracted from `DataModel.mwb`).

### Configuration

Database credentials are currently hardcoded in `app.py`. Update these lines before running:

```python
# app.py
app.config['MYSQL_HOST']     = "localhost"
app.config['MYSQL_USER']     = "root"
app.config['MYSQL_PASSWORD'] = "your_password_here"   # ← change this
app.config['MYSQL_DB']       = "surveyfeedback"
```

> **Recommendation:** Move these to a `.env` file and load with `python-dotenv` (see [Known Issues](#known-issues--security-considerations)).

### Running the App

```bash
python app.py
# → Running on http://127.0.0.1:5000 (debug mode on)
```

---

## Application Pages & Routes

### `/login` — Sign In (`signIn.html`)

Username + password form. On success, stores `user_id`, `user_name`, and the hashed `passcode` in the server-side session and redirects to the homepage. Displays a "Login Failed" error inline on failure.

The sign-in page also has a "Register" button in the navbar that links to `/register`.

---

### `/register` — Registration (`register.html`)

Three-field form: email address, username, password.

**Validation rules:**
- Username must be alphanumeric and at least 7 characters
- Password must be at least 7 characters

On success, the password is SHA-256 hashed, inserted into `users_table`, and the user is immediately logged in and redirected to the homepage.

---

### `/` — Homepage (`index.html`)

Authenticated entry point. Features:
- **Typewriter animation** cycling through "Welcome! :)" and "Get Started!" with a blinking cursor
- **Lottie animation** (local JSON file) displayed to the right
- **Survey code entry form** — enter any numeric survey ID to jump directly to that survey's answering page

---

### `/my_surveys` — Survey Dashboard (`my_surveys.html`)

Lists all surveys owned by the logged-in user as cards. Each card shows:
- Survey title, ID, response count, date created, visibility, and status
- **View Analysis** → `/get_analytics?id=<survey_id>`
- **Delete Survey** → `POST /delete_survey` (with JS confirm dialog)
- **Complete Survey / Make Online** → toggles status between Completed (1) and Online (2)

If no surveys exist, a Lottie animation and a "Get Started" CTA are shown instead.

---

### `/create_survey` — Survey Builder (`create_survey.html`)

A standalone page (does not extend `layout_base.html`) with a dynamic JavaScript-driven form builder.

**Fields:**
- Visibility (Public / Link / Hidden)
- Title
- Description
- Questions (added dynamically)

The question builder supports:
- **Options (Type 1)** — text question + "Add Option" button to dynamically append radio-button choices
- **Free Text (Type 2)** — text question with a textarea for open-ended responses

On "Publish Survey", all state is serialized into a JSON object and POSTed to `/create_survey`. The server calls `InsertSurveyData` and returns the new survey code via `alert()`.

---

### `/get_public_surveys` — Public Survey Browser (`public_surveys.html`)

Lists all surveys with `visibility = 1` (Public) and `status = 2` (Online) as cards showing title, description, and an "Answer Survey" button. Links directly to `/ans_survey?code=<survey_id>`.

---

### `/ans_survey` — Answer a Survey (`ans_survey.html`)

**GET** — Loads the survey by its numeric code. Validates:
- Survey code is numeric
- Survey exists
- Survey is not Completed (status = 1)
- Survey is not Hidden (status/visibility = 3)
- Current user has not already submitted a response

Questions and their options are fetched from the database, sorted by `order`, and rendered dynamically via JavaScript into the page. Multiple-choice questions render radio buttons; free-text questions render textareas.

**POST** — Receives a JSON payload:
```json
{
  "survey_id": 5,
  "ans": [
    { "question_id": 12, "questiontype_id": 1, "optId": 34 },
    { "question_id": 13, "questiontype_id": 2, "val": "Great experience overall" }
  ]
}
```
Inserts a `response` record, then iterates through answers and inserts into `answer` (and `answer_option` for multiple-choice). Returns a success string and redirects to homepage.

---

### `/get_analytics` — Survey Analytics (`analytics.html`)

**Owner-facing** analytics page for a specific survey. For each question:
- **Multiple choice (type 1):** Displays each option and its percentage of total answers (rounded to 2 decimal places), along with total response count
- **Free text (type 2):** Displays all individual text responses in a list

---

### `/logout`

Deletes `user_name`, `passcode`, and `user_id` from the session and renders `errors.html` with a "Logged out successfully" message.

---

### `/about_us`

Static informational page rendered via `about_us.html`.

---

## Route Reference

| Method | Path | Auth Required | Description |
|---|---|---|---|
| `GET/POST` | `/` | ✅ | Homepage — survey code entry |
| `GET/POST` | `/login` | ❌ | Sign in |
| `GET` | `/logout` | ❌ | Destroy session |
| `GET/POST` | `/register` | ❌ | Create account |
| `GET` | `/my_surveys` | ✅ | List owned surveys |
| `GET/POST` | `/create_survey` | ✅ | Survey builder (GET=page, POST=submit JSON) |
| `GET/POST` | `/ans_survey` | ✅ | Answer a survey by code |
| `GET` | `/get_public_surveys` | ✅ | Browse public surveys |
| `GET` | `/get_analytics` | ✅ | View analytics for a survey |
| `POST` | `/delete_survey` | ✅ | Delete a survey (stored procedure) |
| `GET` | `/complete_survey` | ✅ | Toggle survey status Completed ↔ Online |
| `GET` | `/about_us` | ✅ | About page |

---

## Authentication & Session Management

Sessions are stored **server-side on the filesystem** via `flask-session` (`SESSION_TYPE = "filesystem"`). Session files are written to the `flask_session/` directory.

The session stores three keys after login:

| Key | Value |
|---|---|
| `user_id` | Integer primary key from `users_table` |
| `user_name` | Raw username string |
| `passcode` | SHA-256 hex digest of the user's password |

### `authenticate()` Function

Located in `modules/functions.py`. Called at the top of every protected route.

```python
def authenticate(mysql, session):
    # Reads user_name and passcode from session
    # Re-queries the database to verify both still match
    # Returns True if valid, False otherwise
```

This re-validates credentials on **every request** against the database — there is no JWT or token expiry. If a user's password is changed in the database, all existing sessions are immediately invalidated.

### Password Hashing

Passwords are hashed using Python's built-in `hashlib` with SHA-256:

```python
h = hashlib.new("SHA256")
h.update(passcode.encode())
passcode_h = h.hexdigest()
```

> **Note:** SHA-256 without salting is not considered secure by modern standards. See [Security Considerations](#known-issues--security-considerations).

---

## Survey Lifecycle

```
Create Survey
     │
     ▼
status = Online (2), visibility = Public/Link/Hidden
     │
     ├──► Respondents answer (one response per user enforced)
     │
     ├──► Owner views analytics at any time
     │
     ├──► Owner toggles: Complete Survey  → status = Completed (1)
     │                   Make Online      → status = Online (2)
     │
     └──► Owner deletes: CALL DeleteSurvey(id) removes all data
```

**Completion** marks a survey as closed — respondents who attempt to access it receive an error: "This Survey has been completed!"

**Deletion** is permanent and cascades through all related tables via the `DeleteSurvey` stored procedure.

---

## Question Types

| Type ID | Name | Input | Storage |
|---|---|---|---|
| `1` | Options (Multiple Choice) | Radio buttons; exactly one option selected | `answer` row + `answer_option` row with `question_option_id` |
| `2` | Free Text | Textarea | `answer` row with `answer` text column |

Questions are always displayed in ascending `order` within their survey. Options within a question are also sorted by `order`.

---

## Analytics Engine

The `/get_analytics` route computes stats server-side for each question:

**Multiple-choice questions:**
```
percent = COUNT(answer_option where question_option_id = X) / total_responses × 100
```
Rounded to 2 decimal places. `total_responses` = total `answer` rows for that question.

**Free-text questions:**
All `answer` records for that question are fetched and passed to the template as a list of raw text strings.

---

## Helper Module

**File:** `modules/functions.py`

### `authenticate(mysql, session) → bool`

Re-validates the session on every call by running:
```sql
SELECT count(1) FROM users_table WHERE user_name = ? AND passcode = ?
```
Returns `True` if exactly one matching record exists.

### `mysql_db(mysql, query, mode) → varies`

A unified database query helper supporting four execution modes:

| Mode | Behavior | Returns |
|---|---|---|
| `"get"` | `fetchall()` | Tuple of tuples |
| `"fetchone"` | `fetchone()` | Single tuple or `None` |
| `"dict"` | `fetchall()` + column names mapped | List of dicts |
| `"commit"` | `execute()` + `commit()` | `None` |

---

## Frontend Architecture

### Base Layout (`layout_base.html`)

All authenticated pages extend `layout_base.html` using Jinja2 block inheritance. The base template provides:
- Bootstrap 5 CSS/JS via CDN
- A responsive sticky navbar with links to: My Surveys, Public Surveys, About Us, LogOut, and Create Survey
- `{% block main_container %}` — the content injection point

### Dynamic Survey Builder (`create_survey.html`)

JavaScript maintains a `state` array where each element represents one question. On "Add Question":
- If type = Options: creates a label, text input, option container `div`, and "Add Option" button
- If type = Free Text: creates a label and text input only

On "Add Option": appends a new label+input inside the question's option container and pushes to `state[n].opts`.

On "Publish Survey", all state is serialized:
```js
{
  survey_title: "...",
  survey_desc: "...",
  visibility: 1,
  questions: [
    { questionOrder: 1, questionValue: "...", questiontype: 1,
      questionOpts: [{ optOrder: 1, opt_input_value: "..." }, ...] },
    { questionOrder: 2, questionValue: "...", questiontype: 2 }
  ]
}
```
POSTed to `/create_survey` via `fetch()`. A browser `alert()` confirms the survey code.

### Survey Answer Renderer (`ans_survey.html`)

The Jinja2 template passes `data` (including questions and options) as JSON into a `<script>` block:
```js
let data = {{ data | tojson }};
```

JavaScript iterates `data.questions` and dynamically builds radio-button groups (type 1) or textareas (type 2) into `#main_container`. On "Answer Survey", the `state` array is serialized and POSTed to `/ans_survey` as JSON.

---

## Known Issues & Security Considerations

| Issue | Risk | Recommendation |
|---|---|---|
| **Hardcoded DB credentials** in `app.py` | High — credentials visible in source code | Move to `.env` file, load with `python-dotenv`, add `.env` to `.gitignore` |
| **Raw SQL string interpolation** throughout all routes | Critical — SQL injection on every user-controlled input | Use parameterized queries (`cursor.execute(query, (param,))`) or an ORM |
| **SHA-256 without salt** for passwords | High — vulnerable to rainbow table attacks | Replace with `bcrypt` or `argon2` |
| **No CSRF protection** on POST forms | Medium — state-changing forms can be triggered cross-site | Add Flask-WTF CSRF tokens |
| **No authorization check on `/get_analytics`** | Medium — any logged-in user can view any survey's analytics by ID | Verify `survey.user_id == session['user_id']` before rendering |
| **`flask_session/` committed to repository** | Low — exposes session data of past users | Add `flask_session/` to `.gitignore` |
| **`debug=True` in production** | High — exposes interactive debugger | Set `debug=False` or use environment variable |
| **Feedback count always shows 0** on Public Surveys page | Bug — hardcoded `<p>0</p>` in `public_surveys.html` | Query `COUNT(*) FROM response WHERE survey_id = ?` per card |

---

## Future Improvements

| Area | Suggestion |
|---|---|
| **Security** | Parameterized queries, salted password hashing, CSRF tokens |
| **Configuration** | `.env`-based config for DB credentials and `SECRET_KEY` |
| **Analytics** | Chart.js visualizations (pie/bar charts) for multiple-choice breakdowns |
| **Question types** | Rating scales, checkboxes (multi-select), date pickers |
| **Survey editing** | Allow owners to edit title, description, or visibility after creation |
| **Pagination** | Paginate My Surveys and Public Surveys for large datasets |
| **Email notifications** | Notify survey owner when a new response is submitted |
| **Export** | Download survey responses as CSV |
| **Password reset** | Implement the "Forgot Password?" / `/reset_password` route (currently a dead link) |
| **Input validation** | Client-side validation on the survey builder to prevent empty questions/options |
| **Deployment** | Add `requirements.txt`, `Procfile`, or `Dockerfile` for easy deployment to Railway/Render/Heroku |

---

## License

MIT — see `LICENSE` for full terms.
