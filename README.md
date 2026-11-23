# Chronix — exam-aware evening study planner

Chronix turns a student's exam list into a day-by-day evening study timetable. It then checks that timetable against 11 study rules and lists anything it couldn't fit, instead of hiding it.

**Live prototype:** https://claude.ai/artifact/8MetXzGtZhdfSu9djFJ7Bi

SCSE4041 Mini Project — Milestone 2

---

## The problem

Students with several exams in the same few weeks usually plan by hand. Those plans often:
- pack too much into one evening
- skip breaks and revision
- give easy and difficult subjects the same time
- keep scheduling a subject after its exam is over

Chronix builds the plan for them and shows whether it follows the rules.

## What it does

**Input**
- Subjects, each with an exam date and difficulty (Difficult / Moderate / Easy)
- Focused hours per day and the evening window (for example 6:00–10:30 pm)
- Break length, maximum block length and daily revision minutes
- Optional: a weekly rest day, whether exams are held in the day or evening, and syllabus units for each subject

**Output**
- A timetable for every day until the last exam, with exact times and a task for each session
- A short "why" line for each day explaining what was prioritised
- 11 rule checks, each marked pass or fail
- A list of limitations: anything the plan couldn't fit

**Rules it checks**
1. Never go over the daily hours cap (breaks don't count)
2. No overlapping sessions
3. A break between every two sessions
4. No block longer than the maximum (90 min by default)
5. All study inside the evening window
6. Every day after the first opens with revision; Sundays are a weekly review
7. Every session of a difficult subject includes practice
8. A subject stops appearing once its exam has passed
9. Every subject gets time before its exam
10. The evening before an exam is lighter
11. No invented syllabus topics; only units the student pastes are named

## How it works

| Module | What it does |
|---|---|
| Input form | Collects subjects, dates, difficulty and daily settings; saves them in the browser |
| Planner engine | Scores each subject every day as *difficulty ÷ (days to exam)^0.55*, boosted just before its exam, and splits the day's time between the top subjects |
| Session builder | Adds the revision block, splits study into blocks no longer than the maximum, adds breaks and fits the day into the evening window |
| Task assigner | Gives each session a task: first-pass learning → practice → timed practice set → final revision |
| Rule validator | Re-checks the finished plan against the 11 rules and builds the limitations list |
| Timetable view | Day-by-day plan with "why" notes, filters (all days / from today / next 7 days) |
| Progress tracker | Tick sessions off as you finish them; saved in the browser |
| Copy as text | Copies the whole plan for notes or WhatsApp |

The app is a single HTML file with no server, no database and no API key. Everything runs in the browser.

## How to open and run

- **Easiest:** open the live link above.
- **Locally:** download `code/index.html` and open it in any browser. There is nothing to install.
- Click **Load example**, then **Generate plan**. Open the **Rule checks** tab to see validation and limitations.

**Run the tests**
```bash
pip install playwright
python3 code/tests/run_tests.py
```
This opens `code/index.html` in headless Chrome, runs every case in `test-data/test-cases.json` and saves each result to `test-data/outputs/`. If Playwright can't find Chrome, pass its path: `python3 code/tests/run_tests.py /usr/bin/google-chrome`.

## Test results

15 test inputs: 5 normal, 6 edge and 4 invalid. **11 pass, 4 fail.**

| Result | Tests |
|---|---|
| Pass | T1–T11: baseline, late start, rest day, evening exams, syllabus units, tight window, exams only days away, six subjects, two exams on one day, exam tomorrow, one past exam |
| Fail | T12: all exam dates in the past → script error, no plan or message |
| Fail | T13: latest finish before evening start → no plan and no clear error |
| Fail | T14: 0 hours per day → a broken plan is still generated |
| Fail | T15: revision set to 0 → the revision check wrongly reports a failure |

Full table: [`test-data/test-table.md`](test-data/test-table.md). Screenshots of real runs: [`docs/screenshots/`](docs/screenshots/).

## Repository layout

| Path | Contents |
|---|---|
| `prompts/v1/`, `v2/`, `v3/` | Earlier prompt versions, one file per prompt |
| `prompts/final/` | Final prompts |
| `code/index.html` | The app |
| `code/tests/run_tests.py` | Test runner |
| `test-data/` | Test inputs (`test-cases.json`), test table, saved outputs |
| `docs/` | Technical report, architecture diagram, screenshots, Milestone 2 PDF |
| `CHANGELOG.md` | Dated changes since Milestone 1 |

## Known limitations

- Bad input isn't caught: past-only exams crash the page, and impossible times or 0 hours give no clear error (T12–T14).
- Without pasted syllabus units, tasks are generic ("next portion").
- Difficulty weights are hand-tuned and the plan doesn't adapt to how sessions actually went.
- One evening window per day only; data stays in one browser.

**Next (Milestone 3):** input validation with clear error messages, calendar (.ics) export, and re-planning around missed or weak sessions.

## Team

| Name | Enrollment no. | Main task |
|---|---|---|
| Sanskar Srivastava | S24CSEU2577 | prompt design and iteration (v1–final), building the Chronix prototype (planner engine, rule validator, UI) and the GitHub repo |
| Lavanaya Shree | S24CSEU2676 | Research on study-planning methods and existing planner tools; writing and formatting the Milestone 2 PDF and the Technical Report |
| Shrestha Shukla | S24CSEU2671 | Reel script, screen recording of the working demo, voice-over and video editing |
| Arnav Vats | S24CSEU2580 | Designing and running the 15 test cases (normal, edge, invalid), recording expected vs actual results and screenshots of test runs  |

Faculty: Dr. Sonal Kukreja  · Team ID: PE-65-28
