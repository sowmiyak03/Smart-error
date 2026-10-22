# CodeTutor AI - Developer Instructions for AI Agents

## 🎯 Project Overview

**CodeTutor AI** is a Django-based educational platform that provides intelligent code analysis using OpenAI's API. It helps learners understand programming errors through 3-level progressive hints, concept mapping, and multiple correction strategies.

### Architecture
- **Backend**: Django 5.2 with OpenAI GPT-4o-mini integration
- **Frontend**: Single-page HTML/CSS/JavaScript (AJAX-based, no framework)
- **Database**: SQLite (development only)
- **Deployment**: WSGI-compatible (configurable for production)

## 🔑 Critical Components & Data Flow

### Request Flow (Analyze Code)
1. **Frontend** (`coder.html`): User submits code + language selection
2. **API Endpoint** (`tutor/urls.py → analyze_code`): Receives POST request with code
3. **Prompt Engineering** (`views.py:analyze_code`): Constructs comprehensive OpenAI prompt
4. **OpenAI Integration**: Sends prompt to GPT-4o-mini with system role as expert educator
5. **Response Transform** (`transform_response_for_frontend`): Reshapes OpenAI JSON for frontend
6. **Frontend Rendering**: Displays errors, warnings, concept map, and solutions

### Key API Endpoints
| Endpoint | Purpose | Response Format |
|----------|---------|-----------------|
| `/` (GET) | Home page | HTML template |
| `/api/analyze/` (POST) | Full code analysis | JSON with errors, warnings, concept map |
| `/api/analyze/mock/` (POST) | Demo analysis (fallback) | Mock JSON response |
| `/api/get_hint/` (POST) | Progressive hints (L1-L3) | `{"hint": string, "level": int}` |
| `/api/correction-strategy/` (POST) | Multiple fix approaches | `{"strategies": [...]}` |
| `/api/corrected-code/` (POST) | Only corrected code | `{"corrected_code": string}` |

### Response Structure
All endpoints return JSON with consistent error handling. Successful analysis includes:
```python
{
    "analysis_summary": {"total_errors": int, ...},
    "concept_map": [{"category": str, "concept": str, "count": int}],
    "errors": [{"id": int, "type": str, "title": str, "severity": str, "hints": {...}}],
    "warnings": [...],
    "correction_strategies": [{"name": str, "code": str, "pros": [...]}],
    "corrected_code": str,
    "suggestions": [str],
    "best_practices": [str]
}
```

## 🏗️ Project Structure Patterns

### Backend Organization
- `codeTutor/settings.py`: Django config, templates path, installed apps
- `tutor/views.py` (~593 lines): All API logic - analyze, hint, strategy endpoints
- `tutor/urls.py`: URL routing with 5 main endpoints
- `tutor/models.py`: Currently empty (ready for future DB features)
- `tutor/templates/coder.html`: 2358-line frontend (dual-panel layout)

### Frontend Architecture (`coder.html`)
- **Editor Panel**: Code input with syntax highlighting, language selector
- **Analysis Panel**: Tabbed view (Errors, Warnings, Concepts, Suggestions)
- **Dark theme**: Slate-based color scheme (`#0f172a`, `#1e293b`)
- **No external frameworks**: Pure vanilla JS with Fetch API (lines use `@csrf_exempt`)

## ⚙️ Developer Workflows

### Setup & Running
```bash
# Activate virtual environment (Windows)
myenv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set environment variable (required)
set OPENAI_API_KEY=sk-...

# Run development server
cd codeTutor
python manage.py runserver
```

### Key Dependencies
- `Django==5.2.10`: Web framework
- `openai==2.15.0`: GPT-4o-mini API client
- `python-dotenv==1.2.1`: Environment variable loading (`.env` file)
- `pydantic==2.12.5`: Data validation

### Testing & Debugging
- **Mock endpoint** (`analyze_code_mock`): Returns pre-built analysis without API calls (useful for frontend testing)
- **JSON parsing safety**: Response handler cleans markdown code blocks (`\`\`\`json`)
- **Error handling**: Specific catch for OpenAI auth, rate limits, JSON parse errors

## 🔄 Integration Points & Critical Patterns

### OpenAI API Integration
- **Model**: Always use `gpt-4o-mini` (not gpt-4, specified in all requests)
- **Temperature**: 0.2 (analysis) or 0.3 (hints/strategies) - low for consistency
- **Max tokens**: 6000 (analysis), 300 (hints), 1000 (strategies)
- **Error handling**: Check for "authentication", "rate limit", "openai" in error messages
- **API Key**: Required in environment (loaded via `dotenv`)

### Prompt Engineering Standards
1. **System role definition**: Always specify role as expert educator in language
2. **Input structure**: Wrap code in markdown blocks (\`\`\`)
3. **Output format**: Request specific JSON structure with required fields
4. **Educational emphasis**: Prompt explicitly asks for learning-focused guidance
5. **Progressive revelation**: Hints designed to scaffold learning (Nudge→Clue→Solution)

### Response Transformation Rules
- `transform_response_for_frontend()` maps OpenAI fields:
  - `category` → `type`
  - `description` → `description`
  - `cause` → `cause`
  - `fix` → `fix`
  - Separates issues by severity (`error` vs `warning`)

## 📋 Project-Specific Conventions

### Code Analysis Coverage
The `analyze_code` prompt expects:
1. **Error categories**: syntax, logic, performance, security, best_practice, style
2. **Per-issue metadata**: title, description, cause, fix, line/column, code snippet, severity
3. **Concept mapping**: Groups issues by root concepts (e.g., "Indentation", "Type Errors")
4. **Multiple solutions**: 3+ correction strategies with pros/cons for each

### Language Support
Supported languages (specified in `language` param):
- Python, JavaScript, Java, C++, C#

Code language detection from user input; prompt regenerated for each language.

### UI/UX Patterns (Frontend)
- **Dual-panel layout**: Editor left, analysis right
- **Tab-based navigation**: Errors | Warnings | Concepts | Suggestions tabs
- **Severity indicators**: Color-coded (high=red, medium=yellow, low=info)
- **Progressive interactions**: Users click hints to reveal levels 1→2→3
- **Session persistence**: Code stored in browser (`localStorage` implied by "Code remains on refresh")

## 🚀 Common Tasks & Implementation Patterns

### Adding a New Analysis Feature
1. Extend the OpenAI prompt in `analyze_code()` with new analysis requirements
2. Update response JSON schema in prompt (add new top-level field)
3. Add new field to `transform_response_for_frontend()` mapping
4. Create new JS tab/section in `coder.html` to display results
5. Test with mock endpoint first

### Adding a New API Endpoint
1. Create view function in `views.py` with `@csrf_exempt @require_POST` decorators
2. Parse JSON from `request.body`
3. Call OpenAI API with appropriate prompt
4. Return `JsonResponse` with structured output
5. Add route in `tutor/urls.py`

### Handling API Errors Safely
- Check error message for keywords: "authentication", "rate limit", "openai"
- Provide user-friendly messages (don't expose full stack traces)
- Mock endpoint serves as fallback for development/demo
- JSON parsing failures: clean markdown blocks before parsing

## ⚠️ Known Limitations & Gotchas

- **Models.py empty**: Database persistence not yet implemented (currently stateless)
- **No authentication**: All endpoints accept public requests (add auth for production)
- **Frontend-only**: No API documentation beyond code inspection
- **Hardcoded secrets**: `SECRET_KEY` in settings.py (use environment variables for production)
- **SQLite in production**: Current DB setup unsuitable for production (use PostgreSQL)
- **OpenAI dependency**: All analysis requires valid API key; mock endpoint is fallback

## 🔍 Files to Study First

1. [tutor/views.py](../codeTutor/tutor/views.py) - All API logic, prompts, error handling
2. [tutor/urls.py](../codeTutor/tutor/urls.py) - Route definitions, endpoint structure
3. [tutor/templates/coder.html](../codeTutor/tutor/templates/coder.html) - Frontend UI, AJAX calls
4. [codeTutor/settings.py](../codeTutor/codeTutor/settings.py) - Django config, middleware
5. [requirements.txt](../requirements.txt) - Dependencies, versions
