# EcoPulse

> A comprehensive carbon footprint tracking platform for organizations and individuals — combining personalized behavioral tracking, AI-driven recommendations, forecasting, training, and compliance management into one unified system.

---

## Business Problem

Corporate sustainability initiatives often fail to drive meaningful behavior change because they measure emissions at the organizational level without engaging the individuals whose daily choices create them. EcoPulse addresses the following challenges:

### Core Challenges

| Challenge | Description |
|---|---|
| **Disconnection from Personal Impact** | Employees contribute to Scope 3 emissions through commute, diet, office behavior, and travel but rarely see their individual contribution. |
| **Fragmented Carbon Data** | Emissions data lives in spreadsheets, manual surveys, and disconnected tools — no single source of truth exists. |
| **Weak Behavioral Incentives** | Without personalized, actionable feedback loops, sustainability programs fail to change habits at scale. |
| **Employer–Employee Gap** | Companies set carbon reduction goals but lack tools to involve employees in the process or track individual progress. |
| **Regulatory Compliance Pressure** | Frameworks like CSRD, SEC climate disclosures, and other ESG mandates require granular carbon reporting that most organizations lack internally. |

EcoPulse was built to close this gap — transforming individual behavioral data into organizational sustainability outcomes by making carbon awareness **personal, actionable, and measurable**.

---

## Product Thinking

### Why This Exists

Traditional carbon tracking tools focus on top-down reporting: measuring Scope 1, 2, and 3 emissions at a corporate level. What they miss is the **human element** — the daily decisions employees make that collectively drive the majority of Scope 3 emissions. EcoPulse flips this model by starting at the **individual level** and aggregating upward to provide organizational visibility.

### The Dual-Interface Model

EcoPulse is designed as a **two-persona system**:

```
+----------------------------------+     +----------------------------------+
|     EMPLOYEE DASHBOARD           |     |     EXECUTIVE DASHBOARD          |
+----------------------------------+     +----------------------------------+
|  - Carbon footprint tracking     |     |  - Org-wide carbon analytics     |
|  - AI recommendations            |     |  - Compliance tracking           |
|  - Training modules + quizzes    |     |  - Company events (with          |
|  - Peer comparison / leaderboard |     |    emission breakdowns)          |
|  - 12-month emission forecast    |     |  - Company goal management       |
|  - Transaction-based tracking    |     |  - Sustainability tips           |
+----------------------------------+     +----------------------------------+
          |                                         |
          +---------> Aggregated Data <-------------+
```

### Design Decisions

| Decision | Rationale |
|---|---|
| **Two distinct dashboards** | Employees need simplicity, motivation, and gamification. Executives need analytics, compliance, and high-level reporting. Separating concerns keeps each interface focused. |
| **Carbon scoring (0–100)** | Gamifies sustainability. 100 = zero emissions, 0 = 200kg+ CO2e/week. Provides instant feedback on environmental impact without requiring domain expertise. |
| **Dual-mode carbon form** | Recognizes employees contribute carbon through both **workplace behaviors** (commute, office days, video conferencing) and **personal lifestyle** (diet, spending habits). |
| **Heuristic forecasting (4 tiers)** | Conservative (5%), Moderate (10%), Ambitious (20%), Aggressive (30%) — interpretable projections that are actionable and credible, without the opacity of ML black boxes. |
| **Built-in training + quiz** | Education drives adoption. Users learn *why* their actions matter through interactive modules and knowledge checks, then earn certificates. |
| **Peer comparison / leaderboard** | Social proof and healthy competition are proven motivators for sustained behavioral change in workplace settings. |
| **Role-based access via registration** | A single `role` field (`employee` / `executive`) during signup keeps the auth system simple while enabling tailored dashboards. |
| **Transaction-based carbon mode** | Supports personal spending (food, shopping, utilities) as an alternative input, recognizing lifestyle emissions extend beyond the office. |
| **Company Events with emission categories** | Events (off-sites, conferences, meetings) are significant emission sources. Each event tracks travel, venue, accommodation, catering, materials, and digital emissions separately. |

---

## The Flow

### Employee Journey
```
Landing → Login/Register → Dashboard
    │
    ├─→ Carbon Form (Office Mode) ──→ Score & Allocations ──→ Recommendations ──→ Dashboard
    │
    ├─→ Carbon Form (Personal Mode) ──→ Transaction Log ──→ Transaction Footprint ──→ Dashboard
    │
    ├─→ Training Modules ──→ Quiz ──→ Certificate
    │
    └─→ Peer Comparison ──→ Leaderboard
```

### Executive Journey
```
Landing → Login/Register → Executive Dashboard
    │
    ├─→ Company Goals
    │
    ├─→ Compliance Standards
    │
    ├─→ Company Events (Create / Track Emissions)
    │
    └─→ Org-Wide Analytics
```

---

## Technical Architecture

### High-Level Architecture

```
+------------------+
|     Frontend     |  HTML/CSS/JS + Chart.js + Font Awesome
+--------+---------+
         |
         | HTTP Requests
         v
+--------+---------+     +------------------+
|  Flask App       |---->|  PostgreSQL      |
|  (app.py)        |     |  (via SQLAlchemy)|
+--------+---------+     +------------------+
         |
         v
+--------+---------+---------------------------+
|   Blueprints                                  |
|  +-- auth_bp    (login, register, logout)    |
|  +-- employee_bp (dashboard, carbon form,    |
|                  training, quiz, comparison)  |
|  +-- company_bp  (goals, compliance, events) |
+--------+---------+---------------------------+
         |
         v
+--------+---------+     +------------------+
|   Services        |     |  Models          |
|  +-- calculator   |     |  +-- User        |
|  +-- forecast     |     |  +-- CarbonFootp.|
|  +-- recommend    |     |  +-- QuizScore   |
|  +-- quiz         |     |  +-- Training    |
+--------+---------+     |  +-- CompanyGoal |
                         |  +-- CompanyEvent|
                         |  +-- Transaction |
                         +------------------+
```

### Blueprint Structure

| Blueprint | Route Prefix | Purpose |
|---|---|---|
| `auth_bp` | `/` | Landing page, login, register, logout |
| `employee_bp` | `/` | Employee dashboard, carbon form, training, quiz, peer comparison |
| `company_bp` | `/company` | Executive dashboard, company goals, compliance, events |

### Key Modules

#### `carbon_calculator/`
- **`calculator.py`** — `CarbonFootprintCalculator` class with methods for commute, diet, office, and travel emissions. Uses standardized emission factors.
- **`emission_factors.py`** — Reference data for CO2e conversion rates by transport mode, diet type, and office activity.

#### `ai_helpers/`
- **`forecasting.py`** — `CarbonForecaster` generates 12-month projections with 4 reduction scenarios (5%, 10%, 20%, 30%).
- **`recommendations.py`** — `RecommendationEngine` analyzes user data to prioritize improvement areas and serve personalized advice.
- **`quiz.py`** — `SustainabilityQuiz` generates questions, validates answers, and calculates scores.

### Database Models

| Model | Key Fields | Purpose |
|---|---|---|
| `User` | username, email, company, department, role | Auth + organizational context |
| `CarbonFootprint` | commute, diet, office, travel, transaction fields + scores | Per-user emissions tracking |
| `QuizScore` | score, percentage, quiz_date | Training assessment |
| `TrainingProgress` | module_id, completion_percentage, certificate | Learning tracking |
| `SustainabilityTip` | category, tip_text, impact_level | Educational content |
| `CompanyGoal` | name, target_value, deadline | Org-level targets |
| `ComplianceStandard` | requirements, company_progress | Regulatory tracking |
| `CompanyEvent` | travel/venue/accommodation/catering/materials/digital emissions | Event carbon accounting |
| `TransactionData` | category, merchant, amount, carbon_impact | Personal spending tracking |

### Carbon Scoring System

The footprint score is computed on a **0–100 scale**, where **100 = zero emissions** and **0 = 200kg+ CO2e/week**:

- **0 emissions** → Score: 100
- **≥ 200kg CO2e** → Score: 0
- **1–199kg CO2e** → Linear scale with baseline of 100kg/week = Score 50

Formula: `score = 100 - (total_emissions / 100 * 50)`

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Backend Framework** | Flask 3.1+ |
| **Database** | PostgreSQL (via psycopg2-binary) |
| **ORM** | SQLAlchemy 2.0+ |
| **Authentication** | Flask-Login 0.6+ |
| **Web Server** | Gunicorn 23+ |
| **Frontend** | HTML5, CSS3, Vanilla JavaScript |
| **Charts** | Chart.js |
| **Icons** | Font Awesome |
| **Deployment** | uv (Python package manager) |
| **Configuration** | python-dotenv |

---

## Project Structure

```
EcoPulse/
├── app.py                    # Flask app factory, blueprints, db config
├── main.py                   # Entry point (app.run)
├── models.py                 # SQLAlchemy ORM models
├── requirements.txt          # Python dependencies
├── pyproject.toml            # Project metadata + uv lock
├── uv.lock                   # Dependency lockfile
├── routes/
│   ├── __init__.py
│   ├── auth.py               # Login, register, logout, landing
│   ├── employee.py           # Dashboard, carbon form, training, quiz
│   └── company.py            # Executive dashboard, goals, events
├── carbon_calculator/
│   ├── __init__.py
│   ├── calculator.py         # CarbonFootprintCalculator class
│   └── emission_factors.py   # Emission factor reference data
├── ai_helpers/
│   ├── __init__.py
│   ├── forecasting.py        # CarbonForecaster (12-month projections)
│   ├── recommendations.py    # RecommendationEngine
│   └── quiz.py               # SustainabilityQuiz
├── templates/
│   ├── base.html             # Base layout (shared header/nav/footer)
│   ├── landing.html          # Public landing page
│   ├── login.html            # Login form
│   ├── register.html         # Registration form
│   ├── employee/
│   │   ├── dashboard.html
│   │   ├── carbon_form.html
│   │   ├── training.html
│   │   ├── training_module.html
│   │   ├── quiz.html
│   │   └── peer_comparison.html
│   └── company/
│       ├── dashboard.html
│       ├── events.html
│       ├── compliance.html
│       ├── success.html
│       └── cancel.html
├── static/
│   ├── css/
│   ├── js/
│   └── images/
└── attached_assets/          # Uploaded images for UI
```

---

## Installation & Setup

```bash
# Clone the repository
git clone https://github.com/BUVKAUSHIK/EcoPulse.git
cd EcoPulse

# Create a virtual environment
python -m venv venv
source venv/bin/activate  # Linux/macOS
# or
venv\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your DATABASE_URL and SESSION_SECRET

# Initialize the database
flask db upgrade

# Run the development server
python main.py
# or
flask run
```

### Requirements

- **Python:** 3.9+
- **Database:** PostgreSQL (or SQLite for local development)
- **Node:** Optional (for Chart.js, Font Awesome via CDN)

### Production Deployment

```bash
# Using Gunicorn
gunicorn --bind 0.0.0.0:8000 app:app
```

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Follow the codebase conventions (blueprint-based routing, model-first design)
4. Test all affected routes manually and via the relevant template
5. Commit with clear messages (`feat:`, `fix:`, `docs:`, `refactor:`)
6. Push and open a Pull Request

### Adding a New Feature

- Add routes in the appropriate blueprint (`routes/auth.py`, `routes/employee.py`, `routes/company.py`)
- Add models in `models.py` with proper relationships
- Create templates in the matching subdirectory
- Add static assets (CSS/JS) in `static/`
- Update this README if the change affects architecture or setup

---

## License

MIT License — see [LICENSE](LICENSE) for details.
