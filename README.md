# Meepzorp: Multi-Agent Claude Environment

A system enabling multiple specialized Claude agents to collaborate on tasks through the Model Context Protocol (MCP).

## 🌟 Vision

The Meepzorp project creates an environment where:

- Multiple specialized Claude agents collaborate on complex tasks
- Each agent has specific capabilities and knowledge domains
- Agents share information through a standardized protocol
- Documentation can be easily added to enhance agent capabilities
- The system is managed through a well-defined API structure

## 🏗️ Current Implementation

The system follows a modular architecture with these main components:

### Agent Orchestration System

Coordinates communication between specialized agents and manages workflows.

- Central MCP server that routes requests
- Agent registry for capability discovery
- Workflow definition and execution engine
- Context management system

Key Components:
- **Registry System**: Handles agent registration and discovery with capability matching
- **Request Router**: Intelligent routing with retry logic and error handling
- **Workflow Engine**: Multi-step workflow execution with parameter mapping and state management

### Specialized Agent Framework

Provides the foundation for creating domain-specific agents.

- Base agent template for extending
- Capability definition system
- Knowledge base connector
- Inter-agent communication protocol

### Knowledge Repository Integration

Connects to data storage for agent consumption.

- Supabase with pgvector integration
- Placeholder implementation for document retrieval
- Framework for GraphRAG-inspired knowledge representation
- Standardized knowledge access API

## 🚀 Getting Started

### Prerequisites

- Docker and Docker Compose
- Python 3.9+
- Supabase account

### Installation

1. Set up the project directory:
   ```bash
   # You've already created the project directory with the setup scripts
   cd meepzorp
   ```

2. Set up environment variables:
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   # Be sure to add your Supabase URL and API key
   ```

3. Run the setup script:
   ```bash
   chmod +x scripts/setup.sh
   ./scripts/setup.sh
   ```

4. Start the system:
   ```bash
   docker-compose up -d
   ```

5. Access the services:
   - Orchestration API: http://localhost:8000
   - Base Agent: http://localhost:8001
   - Personal Agent: http://localhost:8002

## 🧩 Implemented Components

### Orchestration Service

The orchestration service manages the agent ecosystem, routing requests and handling workflows.

- **Agent Registry**: 
  - Dynamic agent registration with capability descriptions
  - Agent discovery with capability filtering
  - Automatic ID generation and status tracking
  - Backoff retry mechanism for resilience

- **Request Router**: 
  - Capability-based request routing
  - Preferred agent selection
  - Configurable timeouts and retries
  - Comprehensive error handling

- **Workflow Engine**: 
  - Multi-step workflow definition and execution
  - Parameter mapping between steps
  - Variable context management
  - Partial results handling
  - Step retry mechanism

- **API Access**: RESTful API for interacting with the orchestration layer

### Base Agent Framework

A template for creating specialized agents with standardized interfaces.

- **Capability Registration**: System for defining and exposing agent capabilities
- **Communication Protocol**: Standardized methods for agent interaction
- **Knowledge Base Connection**: Framework for accessing knowledge repositories
- **Health Monitoring**: Built-in health check and monitoring endpoints

### Personal Agent Implementation

A specialized agent implementation for accessing knowledge.

- **Search Capability**: Semantic search over knowledge repositories
- **Document Retrieval**: Fetch documents by ID with content filtering
- **Knowledge Graph**: Query relationships between entities in the knowledge graph

### Database Schema

A comprehensive Supabase database schema for storing system data.

- **Agent Registry Tables**: Store agent information and capabilities
- **Workflow Tables**: Define and track workflow executions
- **Knowledge Tables**: Store documents, entities, and relationships
- **Vector Search Functions**: Perform semantic similarity searches

## 📚 Documentation

Documentation is available in the source code. Key files to explore:

- `orchestration/src/registry.py`: Agent registration and discovery
- `orchestration/src/router.py`: Request routing and agent communication
- `orchestration/src/workflows.py`: Workflow definition and execution
- `orchestration/src/main.py`: Core orchestration functionality
- `agents/base/src/*.py`: Base agent framework
- `agents/personal/src/capabilities/*.py`: Personal agent capabilities
- `supabase/migrations/*.sql`: Database schema

## 🔧 Current Limitations

This implementation provides a foundation for a multi-agent system but has some limitations:

- UI management interface is not yet implemented
- Document processing pipeline needs to be fully implemented
- Knowledge base integration uses placeholder implementations
- Additional specialized agents need to be developed
- Database integration for workflows and agent registry is pending

## 🔜 Next Steps

- Implement the UI management dashboard
- Create additional specialized agents for different domains
- Enhance the knowledge base integration with your existing systems
- Develop advanced workflow patterns
- Add comprehensive testing
- Complete Supabase integration for workflow and agent storage
- Implement real-time agent status monitoring
- Add workflow visualization and debugging tools

## 🔗 Related Technologies

- [MCP (Model Context Protocol)](https://github.com/anthropics/anthropic-tools)
- [Supabase](https://supabase.io)
- [pgvector](https://github.com/pgvector/pgvector)
- [FastAPI](https://fastapi.tiangolo.com/)
- [Docker](https://www.docker.com/)