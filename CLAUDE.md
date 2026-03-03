# CLAUDE.md — Meepzorp Development Guide

## Project Overview

Meepzorp is a multi-agent system where specialized Claude agents collaborate through the Model Context Protocol (MCP). The backend is Python/FastAPI, the frontend is Next.js, and everything is orchestrated via Docker Compose with Supabase (PostgreSQL + pgvector) as the database.

## Repository Structure

```
meepzorp/
├── orchestration/          # Central API gateway — agent registry, routing, workflows
│   ├── main.py             # FastAPI app (port 9810)
│   ├── registry.py         # AgentRegistryTool, AgentDiscoveryTool
│   ├── router.py           # RouteRequestTool — capability-based routing
│   ├── workflows.py        # CreateWorkflowTool, ListWorkflowsTool, ExecuteWorkflowTool
│   └── tests/
├── agents/
│   ├── common/             # Shared utilities (auth, LLM, registration)
│   │   ├── auth.py         # get_current_user() dependency
│   │   ├── llm.py          # LLMConfig/LLMService (LangChain: OpenAI, Anthropic, Google)
│   │   └── registration.py # register_agent() for orchestration registration
│   ├── base/               # Agent template (port 8001)
│   ├── personal/           # Knowledge management agent (port 8002)
│   │   └── src/capabilities/  # knowledge.py, graph_suggestions.py
│   ├── task/               # Task management agent (port 8003)
│   │   ├── src/models/task.py  # Task, TaskCreate, TaskUpdate Pydantic models
│   │   └── src/routes.py       # CRUD endpoints
│   ├── document_processor/ # PDF/document analysis agent (port 8004)
│   └── creative_director/  # Creative project management agent
│       ├── creative_director/
│       │   ├── agent.py
│       │   └── capabilities/   # creative_strategy, story_crafter, project_management
│       └── tests/
├── ui/                     # Next.js 14 dashboard (port 3000)
├── supabase/migrations/    # SQL migration files
├── docs/setup/             # Agent setup and port configuration guides
├── docker-compose.yml      # All services definition
├── docker-compose-test.yml # Test configuration
└── requirements.txt        # Root Python dependencies (60 packages)
```

## Tech Stack

| Layer        | Technology                                          |
|------------- |-----------------------------------------------------|
| Backend      | Python 3.9+, FastAPI 0.109+, Uvicorn                |
| Frontend     | Next.js 14.1, React 18.2                            |
| Database     | Supabase (PostgreSQL + pgvector)                     |
| Cache        | Redis                                                |
| LLM          | LangChain (OpenAI, Anthropic Claude, Google GenAI)   |
| Containers   | Docker, Docker Compose                               |
| Data models  | Pydantic v2                                          |
| Task queue   | Celery + Flower                                      |

## Service Ports

| Service             | Default Port |
|---------------------|-------------|
| Orchestration       | 9810        |
| Base Agent          | 8001        |
| Personal Agent      | 8002        |
| Task Agent          | 8003        |
| Document Processor  | 8004        |
| Test Agent          | 9811        |
| UI                  | 3000        |
| Redis               | 6379        |

## Common Commands

### Start all services
```bash
docker-compose up -d
```

### Run Python tests
```bash
# From project root — all tests
pytest

# Specific agent tests
pytest agents/creative_director/tests/
pytest orchestration/tests/

# With coverage
pytest --cov=orchestration orchestration/tests/
```

### Formatting and linting
```bash
black .
isort .
flake8
mypy .
bandit -r .
```

### Database migrations
```bash
# Migrations are in supabase/migrations/ and agents/personal/migrations/
# Apply via Supabase CLI or manually
```

## Architecture & Data Flow

```
Client Request → Orchestration Service (9810)
                      ↓
              Agent Registry (capability lookup)
                      ↓
              Route to appropriate agent
                      ↓
         Agent processes request (8001–8004)
                      ↓
              Response back to client
```

**Agent startup flow:** Each agent registers its capabilities with the orchestration service on boot via `agents/common/registration.py:register_agent()`. The orchestration service stores these in its registry for capability-based discovery and routing.

**Workflow execution:** The orchestration service supports multi-step workflows with parameter mapping between steps, variable context management, and step retry logic.

## Key Conventions

### Python Code
- **FastAPI factory pattern** for all agents — each has a `FastAPI()` app instance
- **Async/await throughout** — all handlers and tools are async
- **Pydantic v2** for request/response models and validation
- **Dependency injection** via FastAPI's `Depends()` for auth, DB access
- **Logging** via `loguru` and `structlog`; never log sensitive data
- **Standard response formats** across all agents

### File Organization
- Each agent is a self-contained module under `agents/`
- Tests live alongside their respective modules (`tests/` inside each agent)
- No duplicate nested directories (avoid `creative_director/creative_director/creative_director`)
- Shared utilities go in `agents/common/`

### API & Communication
- All services expose a `/health` endpoint returning `{"status": "healthy"}`
- MCP protocol for inter-agent communication
- RESTful endpoints with JSON payloads
- API key authentication on all endpoints
- CORS configured on all FastAPI apps

### Database
- Supabase for all persistent storage
- pgvector for semantic search / embeddings
- Row Level Security (RLS) on all tables
- Migrations tracked in `supabase/migrations/`
- Agents access Supabase via REST API client (not direct SQL)

### Docker
- Consistent container naming: `meepzorp-<service-name>`
- Services use port range 8000–8999 (agents) and 9810+ (orchestration)
- Health checks on all services (10s interval, 5s timeout, 3 retries)
- Agent containers mount `agents/common/` for shared code
- Docker network: `mcp-net` (bridge driver)

### Testing
- **pytest** with **pytest-asyncio** (strict mode)
- `@pytest_asyncio.fixture` for async fixtures
- FastAPI `TestClient` for endpoint tests
- `moto` for AWS mocking, `pytest-mock` for general mocking
- Test config often in `pyproject.toml` under `[tool.pytest.ini_options]`

## Environment Variables

Required environment variables (set in `.env` at project root):

| Variable               | Used By                        | Description                  |
|------------------------|--------------------------------|------------------------------|
| `SUPABASE_URL`         | Orchestration, all agents      | Supabase project URL         |
| `SUPABASE_KEY`         | Orchestration, all agents      | Supabase API key             |
| `OPENAI_API_KEY`       | Document processor, LLM service| OpenAI API key               |
| `ORCHESTRATION_PORT`   | All services (default: 9810)   | Orchestration service port   |
| `LOG_LEVEL`            | All services (default: INFO)   | Logging verbosity            |

Agent-specific port variables: `BASE_AGENT_PORT`, `PERSONAL_AGENT_PORT`, `TASK_AGENT_PORT`, `DOCUMENT_PROCESSOR_PORT`, `TEST_AGENT_PORT`, `UI_PORT`, `REDIS_PORT`.

**Never commit `.env` files or API keys.**

## Creating a New Agent

1. Copy `agents/base/` as a template
2. Create a `Dockerfile`, `requirements.txt`, and `src/main.py`
3. Register capabilities using `agents/common/registration.py:register_agent()`
4. Add a `/health` endpoint
5. Add the service to `docker-compose.yml` following existing patterns
6. Use a port in the 8000–8999 range
7. Add tests alongside the agent code
