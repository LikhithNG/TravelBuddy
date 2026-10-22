# 🌍 TravelBuddy – AI-Powered Travel Planning Assistant

[![Python](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.68.0+-00a393.svg)](https://fastapi.tiangolo.com/)
[![React.js](https://img.shields.io/badge/React-18-61DAFB.svg?logo=react&logoColor=white)](https://react.dev/)
[![Docker](https://img.shields.io/badge/Docker-20.10.8+-blue.svg)](https://www.docker.com/)
[![LLM Agents](https://img.shields.io/badge/Agents-LangGraph%20%2B%20GPT--4o-purple.svg)](https://github.com/langchain-ai/langgraph)
[![MCP](https://img.shields.io/badge/Weather-MCP%20Server-orange.svg)](https://modelcontextprotocol.io/)
[![Data Warehouse](https://img.shields.io/badge/Data%20Analytics-Snowflake-29b5e8.svg)](https://www.snowflake.com/)
[![OpenAI](https://img.shields.io/badge/LLM-OpenAI-blueviolet.svg)](https://openai.com/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Documentation](https://img.shields.io/badge/docs-Codelabs-blue.svg)](https://codelabs-preview.appspot.com/?file_id=1kMzJ_qRJrDknPFatF1raPvsoJUatl_-tfJuICo7p4EM#0)

**TravelBuddy** is a next-gen AI travel planner that builds personalized itineraries through natural conversation. Just tell it where you want to go: a team of specialized AI agents finds flights, hotels, restaurants and attractions, checks the weather and travel advisories, and brings everything together into one plan, all in a single chat.

## ✨ Key Features

- 🤖 **Multi-Agent Architecture** – A root agent hands each request to the right specialist: planning, pre-trip preparation or destination discovery.
- 🧠 **Smart Intent Routing** – Fast keyword rules catch common requests instantly, and GPT-4o classifies everything else.
- 🌐 **Real-Time Integrations** – Live data from Amadeus, Google Places, Google Search, OpenWeather and Skyscanner.
- 🌦️ **Weather over MCP** – A Model Context Protocol server answers weather questions for one city or several at once.
- ❄️ **Snowflake Flight Analytics** – Flight fares flow through S3 and Airflow into Snowflake for search and price analysis.
- 🗺️ **Auto-Generated Itineraries** – End-to-end travel plans, exportable as PDFs (coming soon).
- 🔐 **Secure Accounts** – Auth0 sign-in, with every conversation saved so you can pick up where you left off.

## 📊 By the Numbers

| Component | Count |
|---|---|
| Specialist agents behind the root agent | 3 |
| Function-calling tools on the planning agent | 5 |
| Weather MCP tools | 2 |
| REST endpoints (backend + flight service) | 20 |
| PostgreSQL tables | 3 |
| Docker Compose services | 3 |

---
## 🧭 Project Overview

### User Interaction Flow
1. Start a conversation with TravelBuddy.
2. Describe your travel plans, preferences or questions.
3. The root agent routes your request to the right specialist agent.
4. Agents fetch live flights, hotels, places and weather.
5. The itinerary tool assembles everything into a plan.
6. Review your itinerary and keep chatting to adjust it.

## 🧩 Core Features

### 1. Multi-Agent Architecture
- Specialized agents for planning, pre-trip preparation and destination discovery
- A LangGraph root agent coordinates them and answers weather questions directly

### 2. Intent Understanding & Personalization
- Keyword rules route common requests (flights, hotels, food, attractions, packing) with no LLM call
- GPT-4o handles open-ended requests and tailors suggestions to your interests

### 3. Real-Time Integrations
- Connected to Amadeus (flights, hotels), Google Places (restaurants, attractions), Google Custom Search (advisories), OpenWeather and Skyscanner

### 4. Intelligent Itinerary Generation
- Combines results from every tool into one plan
- Delivered as structured JSON, with PDF export coming soon

### 5. Travel Analytics
- Flight fare history stored in Snowflake
- Price analysis for any route and date through `/api/flights/analysis`

---
## 🔬 Methodology

### Data Gathering
- APIs: Amadeus, Google Places, Google Custom Search, OpenWeather, Skyscanner (RapidAPI)
- Responses are normalized and validated as they come in

### Agentic Execution
- `root_agent` receives every message
- Delegates to:
  - `planning_agent` (flights, hotels, restaurants, attractions, itinerary)
  - `pre_travel_agent` (visas, health, storms, travel advisories, packing)
  - `explore_agent` (destination ideas)
  - Weather MCP server (current weather and forecasts)

### Itinerary Assembly
- Collects outputs from the planning tools
- Passes them to the itinerary tool
- Returns the final plan as JSON

### Challenges

- Travel planning is scattered across many sites and apps
- Most tools don't understand what a traveler actually wants
- Building an itinerary by hand takes hours
- Weather and travel advisories are rarely part of the plan

### Opportunities

- Natural language understanding of travel preferences
- Intent-based recommendations instead of keyword search
- One unified travel experience on a single platform
- Learning from user feedback to improve over time

---

## 🏗️ Architecture

### Agent Flow
![TravelBuddy agent flow](https://github.com/user-attachments/assets/afd57572-3403-44e0-b285-9eb53e23ac75)

### MCP Flow
![Weather MCP flow](https://github.com/user-attachments/assets/9172176b-ae8e-42aa-885b-f9a7f0bf16ca)

### User Flow
![TravelBuddy user flow](https://github.com/user-attachments/assets/86233095-7990-4194-9c4f-5303fce3bd28)


## 🔄 Data Pipeline

1. **User Query Ingestion** – Requests start in the React frontend and reach the FastAPI backend.
2. **Intent Parsing & Agent Dispatch** – `root_agent` works out the intent and activates the right agent (planning, pre-travel, explore or weather).
3. **Real-Time API Retrieval** – Each agent calls external APIs (Amadeus, Google Places, OpenWeather and more) for live travel data.
4. **Data Transformation** – Tool modules (`tools/*.py`) normalize and structure the responses.
5. **Analytics Integration** – The flight service stages fares in S3, and an Airflow DAG (`load_to_snowflake`) loads them into Snowflake for trend analysis.
6. **Itinerary Compilation** – The itinerary tool combines the outputs into a step-by-step travel plan.
7. **Response Delivery** – Results return to the UI as structured JSON.

```
Skyscanner (RapidAPI) → flight_service → S3 → Airflow DAG → Snowflake → /api/flights
```

---
## 🔌 Microservices & APIs

| Service              | Purpose                                   | Example Endpoint                            |
|----------------------|-------------------------------------------|---------------------------------------------|
| Agent Chat           | Send a message to the multi-agent system  | `POST /api/llm/ask-question`                |
| Weather (MCP)        | Current weather and forecasts             | MCP tools `get_weather`, `get_current_weather` |
| Flight Search        | Fares for a route and date                | `GET /api/flights/search`                   |
| Flight Analysis      | Price trends for a route                  | `GET /api/flights/analysis`                 |
| Flight Ingestion     | Fetch fares → S3 → Airflow → Snowflake    | `POST /flights/` (flight service)           |
| Chat History         | Save and reopen conversations             | `GET /api/user/chat-history`                |
| User Accounts        | Auth0-backed user profiles                | `GET /api/users/`                           |
| Geocoding            | Turn an address into coordinates          | `GET /api/maps/geocode`                     |

Hotels, restaurants, attractions and itineraries are handled by the planning agent's tools behind `/api/llm/ask-question`. Full API docs are at `http://localhost:8000/docs` once the backend is running.


## 🛠️ Technology Stack

### Backend
- ![Python](https://img.shields.io/badge/Python-FFD43B?style=flat&logo=python&logoColor=blue)
- ![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat&logo=fastapi)
- ![JWT](https://img.shields.io/badge/JWT-black?style=flat&logo=JSON%20web%20tokens)
- ![Auth0](https://img.shields.io/badge/Auth0-EB5424?style=flat&logo=auth0&logoColor=white)
- ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat&logo=postgresql&logoColor=white)

### Frontend
- ![React](https://img.shields.io/badge/React-20232A?style=flat&logo=react&logoColor=61DAFB)
- ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white)
- ![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat&logo=vite&logoColor=white)
- ![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=flat&logo=tailwind-css&logoColor=white)
- ![shadcn/ui](https://img.shields.io/badge/shadcn/ui-gray?style=flat&logo=vercel&logoColor=white)

### AI/ML
- ![OpenAI](https://img.shields.io/badge/OpenAI_GPT--4o-412991?style=flat&logo=openai&logoColor=white)
- ![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=flat&logo=langchain&logoColor=white)
- ![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=flat&logo=langchain&logoColor=white)
- ![MCP](https://img.shields.io/badge/Model_Context_Protocol-FF7E00?style=flat)

### Cloud & Data
- ![AWS S3](https://img.shields.io/badge/AWS_S3-232F3E?style=flat&logo=amazon-aws&logoColor=white)
- ![Snowflake](https://img.shields.io/badge/Snowflake-29b5e8?style=flat&logo=snowflake&logoColor=white)
- ![Apache Airflow](https://img.shields.io/badge/Apache_Airflow-017CEE?style=flat&logo=Apache%20Airflow&logoColor=white)
- ![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=flat&logo=docker&logoColor=white)

### Travel APIs
- ![Amadeus](https://img.shields.io/badge/Amadeus_API-0071C5?style=flat&logo=airfrance&logoColor=white)
- ![Google Maps](https://img.shields.io/badge/Google_Maps_&_Places-4285F4?style=flat&logo=googlemaps&logoColor=white)
- ![OpenWeather](https://img.shields.io/badge/OpenWeather-FF7E00?style=flat&logo=OpenWeatherMap&logoColor=white)
- ![Skyscanner](https://img.shields.io/badge/Skyscanner_(RapidAPI)-0770E3?style=flat&logo=skyscanner&logoColor=white)

---

## 🚀 TravelBuddy Setup Guide

### Prerequisites

1. **Python 3.10+**
   - Verify with: `python --version` or `python3 --version`

2. **Node.js and npm**
   - Node.js 18+ and npm 8+ recommended
   - Verify with: `node --version` and `npm --version`

3. **API Keys**
   - OpenAI API key
   - OpenWeather API key
   - Google API key and Custom Search engine ID
   - Google Maps API key
   - Amadeus API credentials (client ID and secret)
   - Auth0 application (domain, client ID, audience)

4. **Database**
   - PostgreSQL database
   - Snowflake account (for flight search and analytics)

5. **Docker and Docker Compose** (optional)
   - For containerized deployment

### Setup Steps

#### 1. Clone the Repository

```bash
git clone https://github.com/LikhithNG/TravelBuddy.git
cd TravelBuddy
```

#### 2. Set Up Environment Variables

Create a `.env` file in the project root with your credentials:

```
# PostgreSQL Database Configuration
DB_HOST=your-postgres-host
DB_PORT=5432
DB_NAME=travelbuddy
DB_USER=your-username
DB_PASSWORD=your-password
DB_SSL_MODE=require

# Auth0 Configuration
AUTH0_DOMAIN=your-auth0-domain
AUTH0_AUDIENCE=your-auth0-audience

# API Keys
OPENAI_API_KEY=your-openai-api-key
OPENWEATHER_API_KEY=your-openweather-api-key
GOOGLE_API_KEY=your-google-api-key
GOOGLE_CSE_ID=your-google-custom-search-id
GOOGLE_MAPS_API_KEY=your-google-maps-api-key
AMADEUS_CLIENT_ID=your-amadeus-client-id
AMADEUS_CLIENT_SECRET=your-amadeus-client-secret

# Snowflake Configuration
SNOWFLAKE_USER=your-snowflake-user
SNOWFLAKE_PASSWORD=your-snowflake-password
SNOWFLAKE_ACCOUNT=your-snowflake-account
SNOWFLAKE_WAREHOUSE=your-snowflake-warehouse
SNOWFLAKE_DATABASE=your-snowflake-database
SNOWFLAKE_SCHEMA=your-snowflake-schema

# Frontend (also copy these into client/.env for local runs)
VITE_AUTH0_DOMAIN=your-auth0-domain
VITE_AUTH0_CLIENT_ID=your-auth0-client-id
VITE_AUTH0_REDIRECT_URI=http://localhost:3000/auth-callback
VITE_AUTH0_AUDIENCE=your-auth0-audience
VITE_BASE_URL=http://localhost:8000
VITE_GOOGLE_MAPS_API_KEY=your-google-maps-api-key

# Other settings
FRONTEND_URL=http://localhost:3000
PORT=8000
```

The flight service uses its own settings. See [flight_service/README.md](flight_service/README.md).

#### 3. Set Up Backend

1. Create a Python virtual environment:
```bash
python -m venv myenv
```

2. Activate the virtual environment:
   - Windows: `myenv\Scripts\activate`
   - macOS/Linux: `source myenv/bin/activate`

3. Install backend dependencies:
```bash
pip install -r server/requirements.txt
pip install -r weather-mcp/requirements.txt
pip install -r flight_service/requirements.txt
```

4. Create the database tables (one time):
```bash
cd server
python -c "from middlewares.conn_database import engine; from models.userModel import Base as UserBase; from models.llm_chat_historyModel import Base as ChatBase; UserBase.metadata.create_all(bind=engine); ChatBase.metadata.create_all(bind=engine)"
cd ..
```

#### 4. Set Up Frontend

```bash
cd client
npm install
cd ..
```

#### 5. Start the Services

**Option 1: Individual services (development)**

1. Start the weather MCP server:
```bash
cd weather-mcp
CONTAINER_MODE=1 python simplified_mcp_server.py
```

2. Start the backend server (new terminal):
```bash
cd server
python main.py
```

3. Start the flight service (new terminal, optional):
```bash
cd flight_service
uvicorn app.main:app --port 8001
```

4. Start the frontend (new terminal):
```bash
cd client
npm run dev
```

**Option 2: Docker Compose**

```bash
docker compose up --build
```

> Note: `weather-mcp/Dockerfile` is still empty, so add one (or remove that service from `docker-compose.yaml`) before using this option.

#### 6. Access the Application

- Frontend: http://localhost:3000
- Backend API: http://localhost:8000 (docs at `/docs`)
- Weather MCP server: http://localhost:8080
- Flight Service API: http://localhost:8001

---

## 📁 Project Structure

```
TravelBuddy/
├── client/                        # Frontend React application
│   ├── src/
│   │   ├── components/            # UI components
│   │   │   ├── ChatComponent.tsx  # Main chat interface
│   │   │   ├── MapComponent.tsx   # Map visualization
│   │   │   └── WeatherDisplay.tsx # Weather display
│   │   ├── pages/                 # Application pages
│   │   ├── auth/                  # Auth0 provider and protected routes
│   │   └── api/                   # API integration
│   └── package.json               # Frontend dependencies
│
├── server/                        # Main backend server
│   ├── agents/                    # AI agent system
│   │   ├── rootAgent.py           # Main agent coordinator (LangGraph)
│   │   └── prompt.py              # Agent prompts
│   ├── subagents/                 # Specialized AI agents
│   │   ├── planning/              # Flights, hotels, food, attractions, itinerary
│   │   ├── pre_travel/            # Advisories and trip preparation
│   │   ├── explore/               # Destination ideas
│   │   ├── in_travel/             # During-trip help (not yet in the graph)
│   │   └── post_travel/           # After-trip follow-up (not yet in the graph)
│   ├── tools/                     # External API integrations
│   │   ├── weather_tool.py        # Weather MCP client
│   │   ├── flight_search.py       # Amadeus flight search
│   │   └── hotel_search.py        # Amadeus hotel search
│   ├── routes/                    # API routes
│   ├── models/                    # Database models
│   ├── main.py                    # Main server entry point
│   └── requirements.txt           # Backend dependencies
│
├── flight_service/                # Flight data microservice
│   ├── app/
│   │   ├── main.py                # Flight service entry point
│   │   ├── api/                   # Flight API endpoints
│   │   └── services/              # Skyscanner, S3 and Airflow clients
│   └── requirements.txt           # Flight service dependencies
│
├── weather-mcp/                   # Model Context Protocol server for weather
│   ├── simplified_mcp_server.py   # MCP server implementation
│   └── weather_agent.py           # Weather agent implementation
│
├── POC/                           # Early prototypes (CrewAI, Streamlit, Pinecone)
├── docker-compose.yaml            # Docker configuration
└── README.md                      # Project documentation
```

## 📝 Notes

1. **Weather MCP Server**
   - The weather tool talks to the MCP server at `http://weather-mcp:8080` and falls back to `http://localhost:8080`
   - Start the MCP server before the backend so weather questions work

2. **Database Setup**
   - Tables are not created on startup, so run the one-time command in step 3
   - Verify database credentials in the `.env` file

3. **Auth0 Setup**
   - Create an Auth0 application and API in the Auth0 dashboard
   - Add `http://localhost:3000/auth-callback` as an allowed callback URL
   - Put the same domain and audience in both the backend and frontend settings

4. **Flight Service**
   - Set your own `S3_BUCKET_NAME` and `AIRFLOW_API_URL`; the defaults point at the original team's resources

5. **Work in Progress**
   - The `in_travel` and `post_travel` agents are written but not yet connected to the root agent
   - `server/evaluation/` holds an evaluation scaffold whose test data is not included yet
   - Files ending in ` 2` (for example `chatRoute 2.py`) are duplicate copies and are not used

6. **Troubleshooting**
   - API key errors: verify all keys in `.env`
   - Database issues: check that the database is running and reachable
   - MCP server issues: verify it is running on port 8080 and the OpenWeather key is set

## 👥 Contributors

- Shiva Bhargav Bhuvanam (original author, [shivabhargavbhuvanam/TravelBuddy](https://github.com/shivabhargavbhuvanam/TravelBuddy))
- Maintained by [Likhith Nagaralu Gurumurthy](https://github.com/LikhithNG)

---

## 📜 License

MIT License – Feel free to fork, extend, and build upon TravelBuddy.

---
## 🤖 AI Usage Disclosure

AI tools such as OpenAI GPT, LangChain, and GitHub Copilot were used for implementing multi-agent travel assistance, generating responses, and supporting development tasks like code scaffolding and prompt tuning. Claude Code was used to polish this README.
