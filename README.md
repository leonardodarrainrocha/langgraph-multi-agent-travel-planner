# LangGraph Multi Agent Travel Planner

## Project Overview

This project is a production focused Multi Agent AI system built with LangGraph for travel planning. It uses a supervisor based architecture to understand user requests, divide the required tasks, and coordinate specialized agents for flights, hotels, and restaurants.

The system combines LangGraph orchestration, LLM based reasoning, Model Context Protocol (MCP), and SerpAPI to retrieve external travel information.

Unlike a simple linear chatbot, the workflow dynamically decides which specialized agent should be executed and when enough information has been collected to generate the final response.

The project focuses on modular architecture, clear separation of responsibilities, structured state management, external tool integration, and practical Agentic AI design.

## Key Features

* Multi Agent travel planning workflow built with LangGraph
* Supervisor Agent for dynamic task routing
* Specialized Flight Agent
* Specialized Hotel Agent
* Specialized Restaurant Agent
* Response Agent for final answer generation
* LLM based parameter extraction for external tools
* Model Context Protocol (MCP) integration
* SerpAPI integration for external travel data
* Deterministic Python based data extraction and filtering
* Structured state management using Pydantic
* Asynchronous API communication using HTTPX and asyncio
* Modular prompt architecture
* Independent LLM wrapper
* Test suite for agents, nodes, MCP, SerpAPI, and graph execution

## Architecture

The application is organized as a Multi Agent workflow where each specialized agent has a clearly defined responsibility.

**Supervisor Layer**

* Receives the user request
* Understands the current workflow state
* Determines which specialized agent should execute next
* Generates the required agent input
* Decides when the workflow should move to the final response

**Flight Agent**

* Interprets the task received from the Supervisor
* Uses an LLM to generate the required MCP parameters
* Calls the MCP Client
* Receives the raw flight data
* Uses Python to extract and reduce the relevant information
* Stores the processed result in the shared AgentState

**Hotel Agent**

* Interprets the task received from the Supervisor
* Uses an LLM to generate the required MCP parameters
* Calls the MCP Client
* Receives the raw hotel data
* Uses Python to extract and reduce the relevant information
* Stores the processed result in the shared AgentState

**Restaurant Agent**

* Interprets the task received from the Supervisor
* Uses an LLM to generate the required MCP parameters
* Calls the MCP Client
* Receives the raw restaurant data
* Uses Python to extract and reduce the relevant information
* Stores the processed result in the shared AgentState

**MCP Layer**

* Provides a standardized interface between the agents and external tools
* Exposes travel search tools through an MCP Server
* Handles communication with SerpAPI
* Returns the external API response to the corresponding agent

The MCP layer does not perform the business filtering or summarization of the results.

**Data Processing Layer**

The raw responses returned by external APIs can contain a large amount of information that is not required by the following stages of the workflow.

Instead of using another LLM or another agent to summarize this information, each specialized agent uses a deterministic Python extraction function.

For example, the Flight Agent uses a function that extracts relevant information such as airline, flight number, departure, arrival, duration, and price.

The same architecture is applied to hotels and restaurants.

This approach reduces the amount of data stored in the shared state and avoids unnecessary LLM calls, token consumption, and non deterministic filtering.

**Response Layer**

* Receives the information collected by the specialized agents
* Uses the Response Agent to generate the final answer
* Presents the available travel information in a clear and useful format
* Does not perform additional external searches

**LLM Layer**

* Groq API
* LangChain ChatGroq
* LLM based reasoning for routing and parameter generation

**State Management**

* Shared AgentState across the LangGraph workflow
* Structured data validation using Pydantic
* Stores the user request, routing information, and processed travel results

## System Workflow

The workflow follows an iterative execution model.

The user request is first processed by the Supervisor Agent. The Supervisor determines which specialized task should be executed and sends the corresponding input to the selected agent.

The specialized agent generates the required parameters, calls the MCP tool, processes the returned data using Python, and stores the reduced result in the shared state.

The Supervisor then evaluates the updated state and decides whether another specialized agent is required or whether the workflow can move to the Response Agent.

A typical execution can follow this sequence:

User Input → Supervisor Agent → Flight Agent → MCP Client → MCP Server → SerpAPI → Python Data Extraction → Flight Result → Supervisor Agent → Hotel Agent → MCP Client → MCP Server → SerpAPI → Python Data Extraction → Hotel Result → Supervisor Agent → Restaurant Agent → MCP Client → MCP Server → SerpAPI → Python Data Extraction → Restaurant Result → Supervisor Agent → Response Agent → Final Response

The execution is dynamic. The Supervisor does not follow a fixed sequence and can decide which agent should run based on the current state.

## Project Structure

```text
.
├── config.py
├── graph.py
├── main.py
├── requirements.txt
│
├── src/
│   ├── agents/
│   │   ├── flight_agent.py
│   │   ├── hotel_agent.py
│   │   ├── restaurant_agent.py
│   │   ├── response_agent.py
│   │   └── supervisor_agent.py
│   │
│   ├── api/
│   │   └── api.py
│   │
│   ├── auth/
│   │   └── auth.py
│   │
│   ├── llm/
│   │   └── model.py
│   │
│   ├── nodes/
│   │   ├── flight_node.py
│   │   ├── hotel_node.py
│   │   ├── restaurant_node.py
│   │   ├── response_node.py
│   │   └── supervisor_node.py
│   │
│   ├── prompts/
│   │   ├── flight_prompt.py
│   │   ├── flight_result_prompt.py
│   │   ├── hotel_prompt.py
│   │   ├── restaurant_prompt.py
│   │   ├── response_prompt.py
│   │   └── supervisor_prompt.py
│   │
│   ├── schemas/
│   │   ├── agent_state.py
│   │   └── supervisor_response.py
│   │
│   └── tools/
│       ├── mcp_client.py
│       └── mcp_server.py
│
└── test/
    ├── test_ddgs.py
    ├── test_graph_llm.py
    ├── test_langgraph.py
    ├── test_mcp_agent.py
    ├── test_mcp_client.py
    ├── test_mcp_server.py
    ├── test_nodes.py
    └── test_serpapi.py
```

## Tech Stack

* Python
* LangGraph
* LangChain
* Groq
* MCP
* SerpAPI
* Pydantic
* HTTPX
* asyncio
* FastAPI
* Docker

## MCP Integration

The project uses Model Context Protocol to separate the Agentic workflow from external travel services.

The specialized agents do not communicate directly with SerpAPI. Instead, they use an MCP Client to communicate with an MCP Server that exposes the required travel search tools.

This separation allows the external data sources and tools to remain independent from the reasoning and orchestration layers.

The current architecture uses MCP for travel related searches including flights, hotels, and restaurants.

## Data Processing Strategy

A key design decision in this project is the separation between LLM reasoning and deterministic data processing.

The LLM is used when interpretation or reasoning is required, such as understanding the task and generating the parameters required by the MCP tool.

Once the external API returns structured data, Python is used to extract the relevant fields and reduce the result.

This means that the workflow does not introduce an additional agent simply to summarize or filter API responses.

The processing flow is therefore:

External API

MCP Server

Raw Response

Python Extraction

Reduced Result

AgentState

This approach improves efficiency, reduces token usage, and makes the extraction process predictable and reproducible.

## Async Architecture

The application uses asynchronous programming for external communication.

HTTP requests are performed using HTTPX AsyncClient and asynchronous MCP operations are used where applicable.

Synchronous operations can be isolated from the event loop when required, allowing the main asynchronous workflow to continue without unnecessary blocking.

## Testing

The project includes tests covering the main components of the system.

The test suite includes:

* LangGraph workflow execution
* Graph and LLM integration
* Supervisor and node execution
* MCP Server
* MCP Client
* MCP Agent integration
* SerpAPI integration
* External search testing

These tests are used to validate the different layers independently during development.

## Key Design Decisions

* Explicit workflow orchestration using LangGraph
* Supervisor based dynamic routing
* Independent specialized agents for flights, hotels, and restaurants
* Shared AgentState across the workflow
* Structured data validation using Pydantic
* MCP used as the external tool integration layer
* External API communication separated from Agentic reasoning
* Deterministic Python data extraction instead of an additional LLM
* Reduced data stored in the shared state
* Independent prompts for each agent
* Asynchronous external communication
* Modular architecture for future expansion

## Future Extensions

The architecture has been designed to support future Agentic capabilities, including:

* Additional travel agents
* More external MCP tools
* Memory integration
* Human in the loop workflows
* Travel preference management
* Additional external travel providers
* More advanced planning capabilities

## Status

Active development project focused on demonstrating practical Multi Agent and Agentic AI architecture using LangGraph and MCP.

## Author

Leonardo Darrain Rocha
Senior Software Engineer
https://www.linkedin.com/in/leonardo-darrain-rocha-a6062354/
