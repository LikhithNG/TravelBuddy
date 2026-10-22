# TravelBuddy – AI Travel Planning Assistant

[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-005571?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React_18-61DAFB?logo=react&logoColor=black)](https://react.dev/)
[![LangGraph](https://img.shields.io/badge/Agents-LangGraph-purple.svg)](https://github.com/langchain-ai/langgraph)
[![OpenAI](https://img.shields.io/badge/LLM-GPT--4o-412991.svg)](https://openai.com/)
[![MCP](https://img.shields.io/badge/Weather-MCP_server-orange.svg)](https://modelcontextprotocol.io/)
[![Snowflake](https://img.shields.io/badge/Flight_data-Snowflake-29b5e8.svg)](https://www.snowflake.com/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2CA5E0?logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/docs-Codelabs-blue.svg)](https://codelabs-preview.appspot.com/?file_id=1kMzJ_qRJrDknPFatF1raPvsoJUatl_-tfJuICo7p4EM#0)

TravelBuddy plans trips through a chat. You describe where and when you want to go, and a team of AI agents finds flights, hotels, restaurants and attractions, checks the weather and travel advisories, and puts it together into an itinerary.

## At a Glance

| | |
|---|---|
| Specialist agents in the graph | 3 (planning, pre-travel, explore), behind 1 routing agent |
| Planning agent tools | 5 (flights, hotels, restaurants, attractions, itinerary) |
| Weather MCP tools | 2 (`get_weather`, `get_current_weather`) |
| REST endpoints | 20 across the backend and the flight service |
| PostgreSQL tables | 3 (`users`, `chat_sessions`, `chat_messages`) |
| Docker Compose services | 3 (backend, frontend, weather MCP server) |

## How It Works

1. The React app sends your message to `POST /api/llm/ask-question`.
2. The **root agent** (a LangGraph state graph) decides who handles it. Keyword rules catch flights, hotels, food, attractions, itineraries and packing; anything else goes to GPT-4o to classify.
3. Weather questions are answered right away through the weather MCP server, for one city or several at once, without calling the other agents.
4. Everything else goes to one specialist agent, which calls its tools and returns the answer.
5. Chat sessions and messages are stored in PostgreSQL through the chat-history endpoints, so past chats can be reopened.

### Agents

| Agent | Handles | Tools and data |
|---|---|---|
| `root_agent` | Routing every message, answering weather questions | Keyword rules, GPT-4o fallback, weather MCP |
| `planning_agent` | Flights, hotels, restaurants, attractions, full itineraries | Amadeus (flights, hotels), Google Places, itinerary builder |
| `pre_travel_agent` | Visas, health, storms, travel advisories, packing | Google Custom Search |
| `explore_agent` | Destination ideas | Place suggestions |

### Flight Data Pipeline

The flight service fetches fares from the Skyscanner API (through RapidAPI), uploads daily and monthly JSON files to S3, and triggers the Airflow DAG `load_to_snowflake`. The backend then reads the loaded fares from Snowflake for `/api/flights/search` and `/api/flights/analysis`.

```
Skyscanner (RapidAPI) → flight_service → S3 → Airflow DAG → Snowflake → backend /api/flights
```

### Diagrams

**Agent flow**
![TravelBuddy agent flow](https://github.com/user-attachments/assets/afd57572-3403-44e0-b285-9eb53e23ac75)

**MCP flow**
![Weather MCP flow](https://github.com/user-attachments/assets/9172176b-ae8e-42aa-885b-f9a7f0bf16ca)

**User flow**
![TravelBuddy user flow](https://github.com/user-attachments/assets/86233095-7990-4194-9c4f-5303fce3bd28)

## API Endpoints

**Backend (`server/`, port 8000)**

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/llm/ask-question` | Send a chat message to the agent graph |
| POST, GET | `/api/users/` | Create a user, get the signed-in user (Auth0 token required) |
| PUT | `/api/users/` | Update the signed-in user |
| POST, GET | `/api/user/chat-history` | Start a chat session, list sessions |
| GET, DELETE | `/api/user/chat-history/{session_id}` | Get or delete one session |
| POST, GET | `/api/user/chat-history/{session_id}/messages` | Add or list messages in a session |
| GET | `/api/maps/geocode` | Turn an address into coordinates (Google Maps) |
| GET | `/api/flights/search` | Fares for a route and date, from Snowflake |
| GET | `/api/flights/analysis` | Price summary for a route and date, from Snowflake |
| GET | `/api/flights/test` | Sample rows from the flights table |
| POST | `/api/chatmaps` | Placeholder that echoes the message back |
| GET | `/` | Health check |

**Flight service (`flight_service/`)**

| Method | Path | Purpose |
|---|---|---|
| POST | `/flights/` | Fetch fares, upload them to S3 and trigger the Airflow DAG |
| GET | `/flights/search` | Same as above, with query parameters |
| POST | `/flights/test` | Echo the request body to check parsing |
| GET | `/` | Health check |

FastAPI serves interactive docs for each service at `/docs`.

## Tech Stack

- **Agents:** LangGraph, LangChain, OpenAI GPT-4o, Model Context Protocol
- **Backend:** Python, FastAPI, SQLAlchemy, PostgreSQL, Auth0 (JWT)
- **Frontend:** React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui (Radix), TanStack Query, Google Maps
- **Data:** Snowflake, AWS S3, Apache Airflow
- **External APIs:** Amadeus, Google Places, Google Custom Search, Google Maps Geocoding, OpenWeather, Skyscanner (RapidAPI)
- **DevOps:** Docker, Docker Compose

## Setup

### Prerequisites

- Python 3.10+
- Node.js 18+
- PostgreSQL database
- Snowflake account (only for the flight search and analysis endpoints)
- API keys: OpenAI, Amadeus (client ID and secret), Google (API key and Custom Search engine ID), Google Maps, OpenWeather, Auth0

### 1. Clone the repository

```bash
git clone https://github.com/LikhithNG/TravelBuddy.git
cd TravelBuddy
```

### 2. Create a `.env` file in the project root

```
# PostgreSQL
DB_HOST=your-postgres-host
DB_PORT=5432
DB_NAME=travelbuddy
DB_USER=your-username
DB_PASSWORD=your-password
DB_SSL_MODE=require

# Auth0
AUTH0_DOMAIN=your-tenant.auth0.com
AUTH0_AUDIENCE=your-api-identifier

# LLM and travel APIs
OPENAI_API_KEY=
AMADEUS_CLIENT_ID=
AMADEUS_CLIENT_SECRET=
GOOGLE_API_KEY=
GOOGLE_CSE_ID=
GOOGLE_MAPS_API_KEY=
OPENWEATHER_API_KEY=

# Snowflake (flight data)
SNOWFLAKE_USER=
SNOWFLAKE_PASSWORD=
SNOWFLAKE_ACCOUNT=
SNOWFLAKE_WAREHOUSE=
SNOWFLAKE_DATABASE=
SNOWFLAKE_SCHEMA=

# Frontend
VITE_AUTH0_DOMAIN=your-tenant.auth0.com
VITE_AUTH0_CLIENT_ID=
VITE_AUTH0_REDIRECT_URI=http://localhost:3000/auth-callback
VITE_AUTH0_AUDIENCE=your-api-identifier
VITE_BASE_URL=http://localhost:8000
VITE_GOOGLE_MAPS_API_KEY=

# App
FRONTEND_URL=http://localhost:3000
PORT=8000
```

The backend and the weather MCP server read this root `.env`, and so does Docker Compose. For local runs, Vite only reads env files inside `client/`, so copy the `VITE_` lines into `client/.env` as well. The flight service has its own settings; see [flight_service/README.md](flight_service/README.md).

### 3. Install dependencies

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r server/requirements.txt
pip install -r weather-mcp/requirements.txt
cd client && npm install && cd ..
```

Then create the database tables once with the command in [server/README.md](server/README.md#run).

### 4. Run the services (one terminal each)

```bash
cd weather-mcp && CONTAINER_MODE=1 python simplified_mcp_server.py   # port 8080
cd server && python main.py                                          # port 8000
cd client && npm run dev                                             # port 3000
```

Then open http://localhost:3000. The frontend forwards `/api` calls to the backend on port 8000.

To run the flight data pipeline as well, start the flight service on its own port:

```bash
cd flight_service && pip install -r requirements.txt && uvicorn app.main:app --port 8001
```

### Docker Compose

`docker-compose.yaml` defines the backend, frontend and weather MCP server. `weather-mcp/Dockerfile` is empty right now, so write it (or remove that service) before running:

```bash
docker compose up --build
```

## Project Structure

```
TravelBuddy/
├── client/              React + TypeScript frontend (chat, maps, flight search)
├── server/              FastAPI backend
│   ├── agents/          Root agent graph and its prompt
│   ├── subagents/       Specialist agents (planning, pre_travel, explore, in_travel, post_travel)
│   ├── miniagents/      Flight, hotel, restaurant, attraction and itinerary helpers
│   ├── tools/           Amadeus, Google Places, web search, weather and memory tools
│   ├── routes/          API routes
│   ├── controllers/     Request handling
│   ├── models/          SQLAlchemy and Pydantic models
│   ├── middlewares/     Auth0, database and OpenAI connections
│   └── evaluation/      Agent evaluation scaffold
├── flight_service/      FastAPI service: Skyscanner → S3 → Airflow → Snowflake
├── weather-mcp/         Weather MCP server (OpenWeather)
├── POC/                 Early prototypes (CrewAI, Streamlit, Pinecone, Airflow DAGs)
└── docker-compose.yaml
```

## Known Gaps

- `in_travel` and `post_travel` agents are written but not connected to the graph yet, so the router never reaches them.
- `server/evaluation/test.py` comes from Google's ADK travel sample and its test data files are not in the repo, so it does not run.
- `flight_service/app/config/settings.py` defaults to the original team's Airflow server and S3 bucket. Set `AIRFLOW_API_URL` and `S3_BUCKET_NAME` to your own.
- The backend does not create its database tables on startup. See [server/README.md](server/README.md) for a one-line command that creates them.
- Files with ` 2` in the name (for example `chatRoute 2.py`) are duplicate copies and are not used.

## Credits

- Original author: Shiva Bhargav Bhuvanam ([shivabhargavbhuvanam/TravelBuddy](https://github.com/shivabhargavbhuvanam/TravelBuddy))
- This copy is maintained by [Likhith Nagaralu Gurumurthy](https://github.com/LikhithNG)

## License

MIT License. See [LICENSE](LICENSE).

## AI Usage Disclosure

AI tools such as OpenAI GPT, LangChain and GitHub Copilot were used to build the agents, generate responses, scaffold code and tune prompts. Claude Code was used to rewrite this documentation.
