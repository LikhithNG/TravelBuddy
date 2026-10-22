# TravelBuddy Backend

The FastAPI backend for TravelBuddy. It runs the LangGraph agent system, stores users and chat history in PostgreSQL, checks Auth0 tokens, and serves flight data from Snowflake.

## Agent System

`agents/rootAgent.py` builds a LangGraph state graph with GPT-4o:

- **Root agent:** routes each message. Keyword rules catch flights, hotels, food, attractions, itineraries and packing; anything else goes to GPT-4o to classify. Weather questions are answered directly through the weather MCP server (`tools/weather_tool.py`, default `http://weather-mcp:8080`, with `http://localhost:8080` as a fallback).
- **Planning agent** (`subagents/planning/`): 5 tools for flights, hotels, restaurants, attractions and itinerary building, backed by Amadeus and Google Places.
- **Pre-travel agent** (`subagents/pre_travel/`): visas, health, storms and advisories through Google Custom Search.
- **Explore agent** (`subagents/explore/`): destination ideas.

`subagents/in_travel/` and `subagents/post_travel/` are written but not yet added to the graph.

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/llm/ask-question` | Send a chat message to the agent graph |
| POST | `/api/users/` | Create a user |
| GET | `/api/users/` | Get the signed-in user |
| PUT | `/api/users/` | Update the signed-in user |
| POST, GET | `/api/user/chat-history` | Start a chat session, list sessions |
| GET, DELETE | `/api/user/chat-history/{session_id}` | Get or delete one session |
| POST, GET | `/api/user/chat-history/{session_id}/messages` | Add or list messages in a session |
| GET | `/api/maps/geocode?address=...` | Geocode an address with Google Maps |
| GET | `/api/flights/search` | Fares for a route and date, from Snowflake |
| GET | `/api/flights/analysis` | Price summary for a route and date, from Snowflake |
| GET | `/api/flights/test` | Sample rows from the flights table |
| POST | `/api/chatmaps` | Placeholder that echoes the message back |
| GET | `/` | Health check |

Interactive docs are at http://localhost:8000/docs while the server is running.

## Authentication

The frontend signs users in with Auth0 and sends the JWT as `Authorization: Bearer <token>`. `middlewares/auth.py` checks it against `AUTH0_DOMAIN` and `AUTH0_AUDIENCE`.

## Settings

The server reads a `.env` file from `server/` or the project root:

```
DB_HOST=your-postgres-host
DB_PORT=5432
DB_NAME=travelbuddy
DB_USER=your-username
DB_PASSWORD=your-password
DB_SSL_MODE=require

AUTH0_DOMAIN=your-tenant.auth0.com
AUTH0_AUDIENCE=your-api-identifier

OPENAI_API_KEY=
AMADEUS_CLIENT_ID=
AMADEUS_CLIENT_SECRET=
GOOGLE_API_KEY=
GOOGLE_CSE_ID=
GOOGLE_MAPS_API_KEY=
OPENWEATHER_API_KEY=

SNOWFLAKE_USER=
SNOWFLAKE_PASSWORD=
SNOWFLAKE_ACCOUNT=
SNOWFLAKE_WAREHOUSE=
SNOWFLAKE_DATABASE=
SNOWFLAKE_SCHEMA=

FRONTEND_URL=http://localhost:3000
PORT=8000
```

## Run

```bash
cd server
pip install -r requirements.txt
```

The server does not create its tables on startup (that code is commented out in `main.py`, and `middlewares/init_db.py` imports modules that no longer exist). Create them once:

```bash
python -c "from middlewares.conn_database import engine; from models.userModel import Base as UserBase; from models.llm_chat_historyModel import Base as ChatBase; UserBase.metadata.create_all(bind=engine); ChatBase.metadata.create_all(bind=engine)"
```

Then start the server:

```bash
python main.py
```

The API runs at http://localhost:8000. Start the weather MCP server first so weather questions work (see the main [README](../README.md)).

With Docker:

```bash
docker build -t travelbuddy-server .
docker run -p 8000:8000 --env-file ../.env travelbuddy-server
```

## Layout

```
server/
├── agents/          Root agent graph and its prompt
├── subagents/       Specialist agents
├── miniagents/      Flight, hotel, restaurant, attraction and itinerary helpers
├── tools/           Amadeus, Google Places, web search, weather and memory tools
├── routes/          API routes
├── controllers/     Request handling
├── models/          SQLAlchemy and Pydantic models
├── middlewares/     Auth0, database and OpenAI connections
├── prompts/         Travel instructions and keywords
├── scripts/         Sample flight data
├── evaluation/      Agent evaluation scaffold (test data not included)
└── main.py          App entry point
```
