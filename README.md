# AI-Supported Ticket Management

Shared university project for the ZIM IT service desk at the University of Duisburg-Essen. The team built a support assistant that collects a request in a chat, proposes one ITSM ticket category, and stores that category on a [Zammad](https://zammad.com/) ticket. A support agent reviews an open ticket and can keep, change, or clear the category. The model does not send the support reply.

This repository describes the project as a whole. It does not assign individual features to one author.

## Purpose

People who need help from the university IT desk can describe the problem in a chat instead of filing a form by hand. The assistant turns that conversation into a Zammad ticket and proposes a category so the service desk can sort the request. The support team still decides how the open ticket is handled.

## Features

- A Streamlit chat collects the request and can create a Zammad ticket.
- A LangGraph workflow extracts the issue, classifies intent, asks for missing detail, and looks up FAQ or similar material.
- The model must choose one of **Incident**, **Service Request**, **Change**, **Problem**, or **Complaint**. Any other label is stored as **Service Request**.
- The chosen value is written to the Zammad field `kategorie` and copied into the ticket article. On an open ticket, an agent can keep, change, or clear the field. The article stays a snapshot.
- A separate path can close a ticket as solved by the chatbot. Human review of the suggested category happens on tickets that stay open for the service desk.
- Local mail is captured by Mailpit. The model can use a local Ollama fallback or an external API when that key is set in an uncommitted `.env` file.

The category path is drawn in [Ticket triage](docs/architecture.md). A fictional walkthrough is in the [case study](docs/case-study.md).

## Architecture

| Part | Role |
| --- | --- |
| `frontend/` | Streamlit chat, a mock clone for UI checks, and a local stand-in for login |
| `backend/` | FastAPI entry point, LangGraph workflow, retrieval, and Zammad client |
| `zammad/` | Upstream Zammad Compose stack and the category-field bootstrap |
| Mailpit | Local SMTP catcher used by the development stack |

The backend reads secrets only from the environment. `example.env` lists the variable names and placeholders. A real `.env` file is ignored by git.

## Local start

Install Docker with Compose. From the repository root:

```bash
cp example.env .env
```

On Windows PowerShell:

```powershell
Copy-Item example.env .env
```

Fill the placeholders in `.env` on that machine. Leave `ZAMMAD_API_TOKEN`, `SAIA_API_KEY`, and `LANGSMITH_API_KEY` empty until the local Zammad workspace or an optional API is available. Do not commit `.env`.

Start Zammad, the backend, the frontend, and Mailpit:

```bash
docker compose -f docker-compose.yml -f zammad/docker-compose.yml up -d
```

Add the local model as well:

```bash
docker compose -f docker-compose.yml -f zammad/docker-compose.yml -f zammad/scenarios/add-ollama.yml up -d
```

After the stack is up:

| Service | URL |
| --- | --- |
| Zammad | http://localhost:8080 |
| Zammad API | http://localhost:8080/api/v1/ |
| Chat | http://localhost:8501 |
| Backend | http://localhost:8000 |
| Mailpit | http://localhost:8025 |

Create the Zammad admin account in the browser, then create an API token under profile token access and put it in `.env`. Recreate the backend container so it reloads the variable. The first backend start can stay unhealthy while embedding models download. It is ready when the log shows `Application startup complete`.

The bundled Zammad Compose file uses its upstream local development database password. That value is only for this local stack.

## Tests

From the repository root, with the backend and test dependencies installed and `PYTHONPATH` set to the root:

```bash
python -m pytest backend/tests/test_zammad_category.py tests/test_route_after_classification.py tests/test_ticket_category.py -q
```

`tests/test_ticket_category.py` mocks the model. Its regression-fixture check expects `backend/rag/old_tickets.json`. That ticket corpus is not in this repository, so that one check cannot load its file. Live model tests stay behind `RUN_LLM_CATEGORY_TESTS=1` and need an API key in the uncommitted environment. They are not part of the default local check.

## What this copy leaves out

- Real tokens, passwords, and any `.env` file.
- `backend/rag/old_tickets.json`, the ticket corpus used by the regression fixtures.
- Generated Chroma databases, test logs, and Deepeval telemetry.
- The university GitLab CI configuration.

Public FAQ extracts used by retrieval are included. They are knowledge-base text, not support-ticket records.

## License

The Zammad Docker material keeps the upstream GNU Affero General Public License notice in [`zammad/LICENSE`](zammad/LICENSE). The team project did not add a separate license file for the application code.
