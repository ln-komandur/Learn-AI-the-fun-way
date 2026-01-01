# 📋 ALPHA VERSION - Requirements Specification

## Document Purpose

This document describes the requirements that were specified for the Alpha version of the Agentic Job Search Agent. These requirements were provided by the user and implemented in full.

---

## Core Requirements

### 1. No API Key / No Cost Operation

**Requirement:**
- The system must work WITHOUT requiring any paid API key
- Must be completely functional using only local, rule-based logic
- No external service dependencies

**Rationale:**
User wanted to evaluate the system before committing to API costs.

---

### 2. Configuration via Text Files

**Requirement:**
All job search configuration must be provided through text files in a dedicated `setup/` folder:

**Required Files:**
- `urls.txt` - Job board URLs to search (one per line)
- `locations.txt` - Preferred job locations with work preference
- `titles.txt` - Target job titles (one per line)
- `resume.txt` - Complete resume text
- `coverletter.txt` - Cover letter template text
- `linkedin.txt` - Full LinkedIn profile text

**Format for locations.txt:**
```
City, State | work_preference

Examples:
San Francisco, CA | remote
Austin, TX | onsite
Seattle, WA | hybrid
Other locations within USA | remote
```

**Special Case:**
"Other locations within USA | remote" should match any US location if remote.

---

### 3. Interactive Salary Input

**Requirement:**
- System must PROMPT for salary range (minimum and maximum) when starting a search
- Should NOT be in configuration files
- Must accept numeric input

---

### 4. Work Preference Handling

**Requirement:**
- Work preference (remote/onsite/hybrid) specified per location in `locations.txt`
- System prompts for location preference alongside the location itself
- Format: `Location | preference`

---

### 5. Job-by-Job Feedback During Search

**Requirement:**
- System must present each job AS IT IS FOUND
- User provides feedback immediately after seeing each job
- Do NOT wait until the end to ask for all feedback at once

**Workflow:**
```
1. Scrape URL
2. Find Job #1
3. Show Job #1 full details
4. Get user feedback on Job #1
5. Find Job #2
6. Show Job #2 full details
7. Get user feedback on Job #2
... continue for all jobs
```

**Required Information per Job:**
- Full job description
- Company name
- Job title
- Location
- Salary
- Remote status
- **Application URL** (must be prominently displayed)
- Match score
- Match reasoning

---

### 6. URL Display Requirements

**Requirement:**
- Every job MUST show its application URL prominently
- URLs must be easily copy-pasteable
- Job ID alone is NOT sufficient - must show full URL

---

### 7. Application Tracking - Separate Menu

**Requirement:**
- Separate menu option to mark jobs as applied (not just during feedback)
- Ability to mark jobs as applied retroactively
- Track full application lifecycle with multiple stages

**Application Stages:**
1. Initial review
2. Applied
3. Phone screen (scheduled/completed)
4. Interview (scheduled/completed)
5. Offer received
6. Rejected by company
7. Withdrew application

**Menu Options Required:**
- Option to mark jobs as applied
- Option to update application status
- Option to view applied jobs list with current status

---

### 8. URL Search Frequency Control (2-Day Rule)

**Requirement:**
- System must track when each URL was last searched
- If a URL was searched in the last 2 days (48 hours), system should offer to skip it
- Must prompt user: "URL searched X hours/days ago, skip? (Y/n)"
- User can override and search anyway by entering 'n'
- Track per exact URL (not per domain)

**Important:**
- Only successfully scraped URLs count toward the 2-day rule
- If URL fails or has no jobs, don't mark it as "checked" (can check again next day)

---

### 9. Prompt Style Requirements

**Requirement:**
All yes/no prompts must follow this format:

**Capitalized letter = default option**

Examples:
- `(Y/n)` - Default is Yes (pressing Enter = yes)
- `(y/N)` - Default is No (pressing Enter = no)

**NOT allowed:**
- `(yes/no)` - unclear default
- `(Yes/No)` - unclear default

**Accepted inputs:**
- Empty (Enter key) → use default
- y/yes/Y/Yes → yes
- n/no/N/No → no

---

### 10. Job Matching Threshold

**Requirement:**
- System should show top 20% of matches by default
- This threshold must be configurable by user
- User should be able to change this setting

**Example:**
- 10 jobs found, threshold 0.20 → show top 2 jobs (20% of 10)
- 25 jobs found, threshold 0.20 → show top 5 jobs (20% of 25)

**Configuration:**
- Must be stored persistently
- Separate menu option to adjust threshold
- Applied to all future searches

---

### 11. Error Handling

**Requirement for URL failures:**
- If URL fails to scrape: PROMPT user "Retry? (Y/n)"
- Do not automatically skip or retry
- User decides next action

**Requirement for URLs with no jobs:**
- Tell user "No jobs found at this URL"
- Do NOT mark URL as checked (for 2-day rule)
- Can check again next day

---

### 12. Future Enhancement Path

**Requirement:**
- System must be architected to support future API integration
- Same database schema should work with API version
- Same file structure and configuration
- Clear markers in code showing where API would enhance functionality

**Code Markers:**
Comments should indicate: `# API_ENHANCEMENT: [description]`

---

## Workflow Requirements

### Complete Search Workflow:

1. **Startup**
   - Load configuration from `setup/` folder
   - Validate required files exist
   - Connect to database

2. **Begin Search**
   - Prompt for salary range (min/max)
   - Load score threshold from database

3. **Process Each URL**
   - Check 2-day rule
   - If < 2 days: Prompt "Skip? (Y/n)"
   - If skip='n' OR > 2 days: Scrape URL
   - On failure: Prompt "Retry? (Y/n)"
   - If no jobs: Tell user, don't mark as checked

4. **Process Each Job**
   - Parse job details
   - Score against criteria (0-100)
   - Apply learned patterns
   - Filter to top N% (threshold)

5. **Present Each Top Job**
   - Show full details including URL
   - Ask: "Apply/Reject/Maybe? (A/r/m)"
   - Get reasoning: "Why?" (text input)
   - Ask: "Did you apply? (y/N)"
   - Save feedback to database
   - Extract learning patterns
   - Continue to next job

6. **Complete Search**
   - Save run history
   - Display summary statistics

---

## Menu Structure Requirements

**Required Menu Options:**

1. Run job search
2. View all matched jobs
3. Mark jobs as applied
4. Update application status
5. View applied jobs list
6. View statistics
7. Configure score threshold
8. View run history
9. Exit

Each option must be clearly described and easy to access by number.

---

## Data Persistence Requirements

**Must Store:**
- User criteria (from config files + prompted salary)
- All discovered jobs
- User feedback per job
- Application tracking with status history
- URL search history (timestamp per URL)
- Learning patterns extracted from feedback
- Run history (when searches occurred, results)
- Configuration settings (score threshold)

**Database:**
- SQLite database
- Located in `data/` folder
- Created automatically on first run

---

## Learning System Requirements

**Requirement:**
- System must learn from user feedback
- Extract patterns from feedback text
- Store patterns with weights and confidence
- Apply patterns to future job scoring

**Pattern Types:**
- Keyword preferences (e.g., "pytorch" = positive)
- Company size preferences (startup vs enterprise)
- Role type preferences (research vs production)

**Learning Process:**
1. User provides feedback with reasoning
2. System extracts mentioned keywords/concepts
3. Determines sentiment (positive/negative) from context
4. Assigns weight based on action (apply=high, reject=negative)
5. Stores pattern with evidence count
6. Applies pattern weights to future job scores

---

## User Experience Requirements

**Interactive Flow:**
- Clear prompts with examples
- Helpful error messages
- Progress indicators
- Confirmation before destructive actions
- Ability to go back or cancel

**Information Display:**
- Structured, easy to read
- Clear section headers
- Use of emojis for visual scanning
- Important information highlighted

**Feedback Collection:**
- Explain WHY feedback helps
- Provide examples of good feedback
- Make it clear how to provide useful input

---

## Non-Functional Requirements

**Performance:**
- Quick startup (< 1 second)
- Responsive prompts
- Minimal waiting between jobs

**Reliability:**
- Graceful error handling
- No data loss on crashes
- Resume-able operations

**Usability:**
- Self-explanatory prompts
- Comprehensive help documentation
- Clear file organization

**Maintainability:**
- Well-commented code
- Clear function names
- Logical file structure
- Documented API enhancement points

---

## Success Criteria

The system is considered successful if:

✅ Works without any API key
✅ All configuration in text files
✅ Prompts for salary interactively
✅ Shows jobs one-by-one with full details including URLs
✅ Collects feedback per job as found
✅ Tracks applications through full lifecycle
✅ Implements 2-day URL rule with override
✅ Uses Y/n prompt style with defaults
✅ Configurable score threshold
✅ Clear path to API upgrade
✅ Comprehensive documentation
✅ Production-ready code quality

---

## Out of Scope

The following were explicitly NOT required for Alpha version:

❌ Real web scraping (mock data acceptable for demo)
❌ API integration
❌ Web interface
❌ Email notifications
❌ Calendar integration
❌ Multi-user support
❌ Cloud deployment

These may be added in future versions.

---

## Implementation Notes

**File Structure:**
```
job_agent_alpha/
├── setup/               # User configuration files
│   ├── urls.txt
│   ├── locations.txt
│   ├── titles.txt
│   ├── resume.txt
│   ├── coverletter.txt
│   └── linkedin.txt
├── data/                # Database (created automatically)
│   └── alpha_agent.db
├── alpha_cli.py         # Main interface
├── alpha_agent.py       # Core logic
├── alpha_database.py    # Data persistence
├── alpha_tools.py       # Scraping & matching
└── README_ALPHA.md      # Documentation
```

**Technology Stack:**
- Language: Python 3.10+
- Database: SQLite
- Dependencies: None (uses standard library)
- Platform: Linux/Unix (Ubuntu primary)

---

## Validation Checklist

To verify all requirements were met:

- [x] No API key required for operation
- [x] Configuration files in setup/ folder
- [x] Salary prompted during search
- [x] Work preference in locations.txt format
- [x] Job-by-job feedback (not batched at end)
- [x] Full job details shown including URL
- [x] Separate menu for application tracking
- [x] Track application status through stages
- [x] 2-day URL rule with prompt to override
- [x] Y/n prompt style with capitals for defaults
- [x] Per-URL tracking (not per-domain)
- [x] Configurable score threshold
- [x] Retry prompt on URL failure
- [x] Don't mark failed URLs as checked
- [x] Learning from feedback
- [x] Clear API enhancement markers in code
- [x] Persistent database storage
- [x] Complete menu structure (9 options)
- [x] Comprehensive documentation

---

## Version

**Alpha v1.0** - Initial release implementing all specified requirements

**Status:** ✅ Complete - All requirements implemented and tested

---

**This document represents the exact requirements as specified by the user for the Alpha version of the Agentic Job Search Agent.**
