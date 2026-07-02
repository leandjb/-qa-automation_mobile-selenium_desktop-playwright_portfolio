# QA Automation Portfolio

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/pytest-8.3.4-0A9EDC?logo=pytest&logoColor=white" alt="pytest">
  <img src="https://img.shields.io/badge/Allure-Report-FF6B35" alt="Allure">
  <img src="https://img.shields.io/badge/Playwright-1.40-2EAD33?logo=playwright&logoColor=white" alt="Playwright">
  <img src="https://img.shields.io/badge/Selenium-4.0-43B02A?logo=selenium&logoColor=white" alt="Selenium">
  <img src="https://img.shields.io/badge/CI-GitHub%20Actions-2088FF?logo=githubactions&logoColor=white" alt="CI">
</p>

<p align="center">
  This is a implementation of API + UI automated testing framework targeting public sites covering Mobile emulation (Selenium) and Desktop (Playwright). Pattern Design was POM - Page Object Model with Allure reporting, email and push notifications (Slack) and Allure test report.
</p>

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Test Coverage](#test-coverage)
- [Architecture](#architecture)
- [Key Design Decisions](#key-design-decisions)
- [Reporting System](#reporting-system)
- [Scheduling & Notifications](#scheduling--notifications)
- [Environment Setup](#environment-setup)
- [Running Tests](#running-tests)
- [Development Guidelines](#development-guidelines)

---

## Tech Stack

| Category | Tool |
|----------|------|
| **Language** | Python 3.12 or higher |
| **UI Automation (Mobile)** | Selenium WebDriver + Chrome Mobile Emulation (iPhone) |
| **UI Automation (Desktop)** | Playwright - Chromium |
| **API Testing** | requests  |
| **Test Framework** | pytest + pytest-xdist (Concurrency) |
| **Architecture Pattern** | POM - Page Object Model  |
| **Reporting** | Allure Report + custom HTML Email report + historical trend table |
| **Notifications** | Slack Webhook & Gmail SMTP |
| **Cloud Archival** | Google Drive API |
| **Environment Management** | python-dotenv |

---

## Test Coverage

This portfolio targets two different public demo sites, simulating a real product's test layering strategy:

### 📱 Mobile — [saucedemo.com](https://www.saucedemo.com) (Selenium + Phone Emulation)

| Test ID | Category | Test |
|---------|----------|------|
| M-LOG-001 | Login | Standard user login → product page |
| M-LOG-002 | Login | Locked-out user → error message validation |
| M-LOG-003 | Login | Wrong password → generic error (does not leak account existence) |
| M-INV-001 | Inventory | Mobile product page lists correct quantity |
| M-INV-002 | Inventory | Sort price low→high validates ascending order |
| M-CHK-001 | Checkout | Add to cart badge number updates |
| M-CHK-002 | Checkout | Full checkout flow (login → add items → fill form → confirm) |
| M-NAV-001 | Navigation | Mobile menu toggle, navigation items validation |

### 🖥️ Desktop — [reqres.in](https://reqres.in) (Playwright + API)

| Test ID | Category | Test |
|---------|----------|------|
| USR-001 | Users | List users (paginated) + JSON Schema validation |
| USR-002 | Users | Get single user |
| USR-003 | Users | Get non-existent user → 404 |
| USR-004 | Users | Create user → 201 + id + createdAt |
| AUT-001 | Auth | User registration success → id + token |
| AUT-002 | Auth | Registration missing password → 400 |
| AUT-003 | Auth | User login success → token |
| AUT-004 | Auth | Login unknown user → 400 |
| RBS-001 | Robustness | Response time < 3 seconds |
| RBS-002 | Robustness | Content-Type validation |
| RBS-003 | Robustness | GET idempotency validation |
| RBS-004 | Robustness | Invalid id graceful degradation (no 5xx) — parametrized 3 cases |

---

## Architecture

```
qa-automation-portfolio/
├── mobile_pack/                      # 📱 Mobile module (Selenium)
│   ├── pages/                        # POM: mobile page objects
│   │   ├── base_page.py              # Shared WebDriverWait dynamic wait wrapper
│   │   ├── login_page.py
│   │   ├── inventory_page.py         # Includes hamburger menu toggle
│   │   └── checkout_page.py          # CartPage / CheckoutPage / CheckoutCompletePage
│   ├── tests/
│   │   ├── api/                      # API layer (reserved)
│   │   └── ui/                       # Selenium iPhone X Emulation tests
│   │       ├── test_login.py         # M-LOG-001/002/003
│   │       ├── test_inventory.py     # M-INV-001/002
│   │       └── test_checkout.py      # M-CHK-001/002, M-NAV-001
│   └── conftest.py                   # mobile_driver fixture, auto screenshot on failure
│
├── web_pack/                         # 🖥️ Desktop module (Playwright)
│   ├── pages/                        # POM: desktop page objects
│   │   ├── base_page.py              # Shared Playwright dynamic wait wrapper
│   │   ├── login_page.py
│   │   ├── inventory_page.py
│   │   └── checkout_page.py          # CartPage / CheckoutPage / CheckoutCompletePage
│   ├── tests/
│   │   ├── api/                      # API layer tests (reqres.in)
│   │   │   ├── test_users.py         # USR-001/002/003/004
│   │   │   ├── test_auth.py          # AUT-001/002/003/004
│   │   │   └── test_robustness.py    # RBS-001/002/003/004
│   │   └── ui/                       # Playwright Chromium tests
│   │       ├── test_login.py         # UI-LOG-001/002/003
│   │       ├── test_inventory.py     # UI-INV-001/002/003
│   │       └── test_checkout.py      # UI-CHK-001/002
│   └── conftest.py                   # driver / reqres fixture, auto screenshot hook
│
├── utils/                            # Shared utilities
│   ├── api_session.py                # Build requests session from Playwright cookies
│   └── report/
│       ├── send_report.py            # HTML Email report generation, Drive upload, Slack push
│       ├── inject_trend_table.py     # Cross-run historical trend table (rolling 30 entries)
│       ├── patch_trend_dates.py      # Allure trend chart epoch timestamp fix
│       └── gdrive_auth.py            # Google Drive one-time OAuth2 authorization
│
├── config/                           # Environment configuration
│   ├── settings.py                   # Frozen dataclass, single source of truth
│   └── credentials.py                # .env notification credential reader
│
├── data/                             # External test data (JSON)
│   └── users.json                    # Accounts, checkout info (no hardcoding)
├── categories.json                   # Allure four-tier classification rules
├── run_tests_scheduled.ps1           # Scheduled script (tests → Allure → report → notification)
├── .github/workflows/ci.yml          # GitHub Actions CI
├── pytest.ini
├── requirements.txt
└── .env.example                      # Environment variable template (no secrets)
```

---

## Key Design Decisions

| Decision | Description |
|----------|-------------|
| **Dual-engine UI** | Mobile uses Selenium Chrome Mobile Emulation to simulate real device behavior; Desktop uses Playwright with built-in auto-wait to reduce flakiness |
| **API ↔ UI cross-validation** | After login, cookies are extracted from the browser to build a shared session, keeping API tests consistent with UI state |
| **Strict POM** | All selectors live only in `pages/`, test files have zero locators, reducing maintenance cost |
| **Dynamic waits** | No `sleep()` fixed waits — always use state-based dynamic waits |
| **Allure three-tier classification** | `categories.json` separates "skipped due to environment constraints" from "real bugs", reducing false positives |
| **Data externalization** | Accounts and input values loaded from `data/` JSON, secrets injected via `.env`, no hardcoding |
| **State isolation** | Each test gets a function-scoped driver, supports `-n auto` parallel execution, tests never pollute each other |

---

## Reporting System

### Allure Report
- Automatic screenshots attached to Allure report on UI test failure or skip
- `categories.json` separates "environment skip" from "real bug" for quick triage
- Each run automatically merges historical trends, supporting pass rate tracking across runs

### HTML Email Report (`send_report.py`)
- Accordion-style result expansion with inline failure screenshots (CID inline)
- Cross-run **historical trend table** (`inject_trend_table.py`)
- Test environment info, execution time, per-feature pass rate overview
- Full Allure report auto-uploaded to Google Drive with share link in email

### Slack Notifications
- Automatic summary push to designated channel after execution (pass rate, failure count, Drive link)

---

## Scheduling & Notifications

`run_tests_scheduled.ps1` completes the full pipeline in one go:

```
Mobile UI tests → Desktop UI tests → API tests
    → Merge Allure history
    → Generate Allure HTML report
    → Archive to HistoryReports/
    → Send HTML Email report
    → Upload to Google Drive
    → Push Slack notification
```

Can be scheduled via Windows Task Scheduler (e.g., daily at 06:00).

---

## Environment Setup

### 1. Install dependencies

```bash
# Add Scoop main bucket
scoop bucket add main

# Install Python and Allure via Scoop
scoop install main/python
scoop install main/allure

# Install project dependencies
pip3 install -r requirements.txt
playwright install chromium --with-deps
```

### 2. Configure environment variables

```bash
cp .env.example .env
```

`.env.example` key variables (public demo accounts, ready to run):

```env
# UI test target
SAUCE_BASE_URL=https://www.saucedemo.com
SAUCE_USER_STANDARD=standard_user
SAUCE_PASSWORD=secret_sauce

# API test target
REQRES_BASE_URL=https://reqres.in/api

# Browser mode
BROWSER_MODE=headless

# Report notifications (optional)
REPORT_SENDER_EMAIL=
REPORT_SENDER_PWD=
REPORT_RECEIVER_EMAIL=
SLACK_WEBHOOK_URL=
GDRIVE_FOLDER_ID=
```

> When notification variables are left empty, Email / Slack / Drive features are automatically skipped without affecting test execution.

### 3. Google Drive authorization (optional)

```bash
python utils/report/gdrive_auth.py
```

---

## Running Tests

```bash
# Run all tests
pytest

# Run Mobile UI only
pytest mobile_pack/tests/ui -m ui

# Run Desktop API only
pytest web_pack/tests/api -m api

# Run smoke tests (core paths)
pytest -m smoke

# Run negative tests only
pytest -m negative

# Parallel execution
pytest -n auto

# Generate Allure report
pytest --alluredir=allure-results
allure generate allure-results -o allure-report --clean
allure open allure-report

# Preview Email report (no actual send)
python utils/report/send_report.py --preview
```

---

## Supported Test Markers

| Marker | Purpose |
|--------|---------|
| `api` | All API tests |
| `ui` | All UI tests (requires browser) |
| `mobile` | Mobile emulation tests (Selenium) |
| `web` | Desktop tests (Playwright) |
| `smoke` | Core paths, must pass on every commit |
| `negative` | Negative / error path tests |

---

## Development Guidelines

- No selectors directly in test files — must go through POM
- No `sleep()` fixed waits — Mobile uses dynamic wait wrapper, Desktop uses Playwright built-in waits
- No JavaScript injection to bypass UI interactions
- No hardcoded credentials — must come from `.env`
- Every test case must include Test ID, steps, and expected result (in docstring)
- Flaky tests must be root-caused, never masked with retries
- Shared logic like report status determination follows DRY — write once
