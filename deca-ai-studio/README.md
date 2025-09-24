# DECA AI Studio - Architecture Study

This directory contains comprehensive architecture documentation and diagrams for the DECA AI Studio project.

## Contents

### 📊 [Architecture Diagram](./architecture-diagram.md)
Visual representations of the system architecture including:
- **System Architecture Diagram**: High-level component overview with Mermaid diagrams
- **Data Flow Diagram**: Sequence diagrams showing request/response flows
- **Component Interaction Matrix**: Service communication patterns
- **Infrastructure Deployment View**: AWS cloud infrastructure layout
- **Message Flow Patterns**: NATS messaging patterns and flows

### 📖 [Architecture Documentation](./architecture-documentation.md)
Comprehensive technical documentation covering:
- **System Overview**: Purpose, capabilities, and key features
- **Architecture Principles**: Microservices, event-driven, polyglot design
- **Technology Stack**: Detailed breakdown of languages, frameworks, and tools
- **Core Components**: In-depth analysis of all 11 major services
- **Data Architecture**: MongoDB collections, Redis patterns, data flows
- **Communication Patterns**: HTTP/REST and NATS messaging protocols
- **Flow Execution Engine**: Workflow orchestration and node execution
- **Security Architecture**: Authentication, authorization, and data protection
- **Deployment Architecture**: AWS ECS, containers, and infrastructure
- **Scalability and Performance**: Scaling strategies and optimization
- **Development Workflow**: Monorepo structure, CI/CD, testing
- **Monitoring and Observability**: Logging, metrics, and alerting

## Key Insights

### System Architecture
DECA AI Studio is built as a **microservices platform** using:
- **11 Core Services**: API Server, Flow Manager, Node Runner, Agent Server, etc.
- **Polyglot Implementation**: Go for performance-critical services, Python for AI/ML
- **Event-Driven Communication**: NATS JetStream for reliable messaging
- **React Frontend**: Modern TypeScript-based user interface

### Technology Highlights
- **Go Services**: High-performance backend services using Gin framework
- **Python AI Services**: FastAPI-based services for AI agent management
- **MongoDB + Redis**: Primary database with distributed caching
- **AWS ECS/Fargate**: Cloud-native containerized deployment
- **Auth0**: Enterprise authentication and authorization

### Core Capabilities
- **Visual Flow Designer**: No-code/low-code AI workflow creation
- **35+ Node Types**: Extensive library of processing nodes
- **Real-time Processing**: Webhook, schedule, and event triggers
- **Multi-tenant**: Workspace isolation and security
- **AI Agent Management**: Multiple AI providers and custom agents

## Architecture Patterns

### Microservices
- **Domain-Driven Design**: Services organized by business capability
- **Independent Deployment**: Each service can be updated independently
- **Technology Diversity**: Right tool for each service's needs

### Event-Driven
- **Asynchronous Communication**: Services communicate via events
- **Loose Coupling**: Services don't need direct knowledge of each other
- **Scalable Processing**: Events processed in parallel

### Cloud-Native
- **Containerization**: All services run in Docker containers
- **Auto-scaling**: Services scale based on demand
- **High Availability**: Multi-AZ deployment with load balancing

## Data Flow Summary

1. **Client Request** → API Server (Authentication/Validation)
2. **Flow Execution** → Flow Manager (Orchestration)
3. **Node Processing** → Node Runner (Individual step execution)
4. **AI Operations** → Agent Server (AI model interaction)
5. **Custom Logic** → Function Server (User-defined functions)
6. **Event Processing** → Event Dispatcher (Event routing)
7. **Background Tasks** → Task Runner (Async operations)

## Security Overview

- **Auth0 Integration**: Centralized authentication
- **JWT Tokens**: Secure service communication
- **Role-Based Access**: Granular permissions system
- **Multi-tenant Isolation**: Workspace data separation
- **Encryption**: TLS in transit, AES-256 at rest
- **AWS KMS**: Key management and rotation

## Deployment Architecture

- **AWS ECS/Fargate**: Serverless container orchestration
- **Application Load Balancer**: Traffic distribution
- **MongoDB Atlas**: Managed database with replica sets
- **ElastiCache Redis**: Managed caching layer
- **CloudWatch**: Monitoring and logging
- **ECR**: Container image registry

This architecture enables DECA AI Studio to provide a robust, scalable, and secure platform for building and managing AI-powered applications and workflows.

## Generated Analysis
- **Date**: 2024-09-24
- **Source**: DECA AI Studio monorepo analysis
- **Tools Used**: Claude Code architecture analysis
- **Coverage**: Complete system architecture and all major components