# Architecture — MindDesk

This document provides a high-level overview of the MindDesk architecture, separating the conceptual design from its current technical implementation. 

For an in-depth technical analysis of the Agentic execution, RAG pipeline, and engineering trade-offs, refer to the **[System Design Document](SYSTEM_DESIGN.md)**.

---

## Table of Contents

- [High-Level System Architecture](#high-level-system-architecture)
- [Component Dependency Map](#component-dependency-map)
- [Hardware Integration Architecture](#hardware-integration-architecture)
- [Current Reference Implementation](#current-reference-implementation)

---

## High-Level System Architecture

MindDesk is a modular, multi-modal AI platform built on a distinct **five-tier architecture**:

1. **Frontend Layer:** A responsive client application providing modular interfaces for Chat, Administration, Surveillance, and Task Scheduling. It communicates via RESTful HTTP calls and Server-Sent Events (SSE) for streaming responses.
2. **API Layer:** The central routing hub. It enforces Role-Based Access Control (RBAC), manages sessions, and routes domain-specific requests to backend services.
3. **Agent Engine:** The orchestration brain. Instead of simple query-response loops, it decomposes user requests into directed task graphs, manages a shared execution context, and implements an autonomous LLM-driven retry loop for error recovery utilizing a planning model.
4. **Service & Tool Modules:** Specialized modules mapping directly to system capabilities (e.g., Document retrieval, Edge Device communication). These act as execution boundaries for the Agent Engine.
5. **Data & Storage Layer:** A polyglot persistence strategy enforcing strict data isolation between relational databases, document databases, and vector storage layers.

```mermaid
flowchart TD
    subgraph Client["Frontend Layer"]
        UI[Responsive SPA]
    end

    subgraph API["API Layer"]
        Routes[API Routes: Auth, Chat, Scheduler, Devices, Surveillance, Uploads]
    end

    subgraph Agent["Agent Engine"]
        MC[Mode Classifier]
        PL[Planner]
        EX[Executor]
        SM[State Manager]
        TR[Tool Registry]
        
        MC --> PL
        PL --> EX
        EX <--> SM
        EX --> TR
    end

    subgraph Tools["Tool Modules / Services"]
        CT[Chat Tools]
        DT[Document Tools]
        VT[Vision Tools]
        PT[Productivity Tools]
        WT[Web Tools]
        IoT[Device Control]
    end

    subgraph Storage["Data Persistence"]
        Rel[(Relational DB)]
        Doc[(Document DB)]
        Vec[(Vector Storage Layer)]
    end

    Client -- HTTP/SSE --> API
    API --> Agent
    TR --> Tools
    Tools --> Storage
    API --> Storage
```

> [!NOTE]
> Current implementation technologies are documented separately below.

---

## Component Dependency Map

The backend is modularized to ensure components remain decoupled. The Agent Engine orchestrates logic, while the Tool Modules handle specialized execution and storage interaction.

```mermaid
graph LR
    subgraph "Entry Points"
        GP[Chat API]
        UP[Upload API]
        DP[Device API]
        SP[Scheduler API]
        SVP[Surveillance API]
    end

    GP --> CORE[Agent Engine]
    CORE --> MC2[Mode Classifier]
    CORE --> PL2[Planner]
    CORE --> EX2[Executor]
    
    EX2 --> SCR[State Manager]
    CORE --> TOOLS[Tool Registry]
    TOOLS --> MODULES[Service Modules]
    
    MODULES --> DH[Document Handler / RAG]
    MODULES --> DA[Tabular Analysis Engine]
    MODULES --> CD[Edge Device Controller]
    
    DH --> MDB[Document Database]
    DH --> VDB[Vector Storage Layer]
    DP --> PG_DEV[Relational: device_db]
    SP --> PG_SCH[Relational: scheduler_db]
    SVP --> PG_SURV[Relational: surveillance_db]
```

> [!NOTE]
> Current implementation technologies are documented separately below.

---

## Hardware Integration Architecture

MindDesk bridges software and hardware through dedicated interfaces, ensuring the core agent logic is unaware of physical hardware nuances.

- **Computer Vision Hardware:** Processed locally by the host machine to maintain privacy and reduce latency. The pipeline captures local camera feeds, which are processed by an Object Detection Model and a Face Recognition Model.
- **Edge Device Control:** External physical actions are routed through a dedicated edge microcontroller via serial communication, triggering physical relays based on string commands dispatched by the system.

---

## Current Reference Implementation

While MindDesk's architecture describes modular roles, the current reference implementation uses the following specific technologies:

- **Frontend Layer:** React 18 SPA, Tailwind CSS
- **API Layer:** Python Flask
- **Agent Planning Model:** Llama 3.1 (via Ollama)
- **Object Detection Model:** YOLOv11
- **Face Recognition Model:** InsightFace
- **Relational Storage:** PostgreSQL
- **Document Storage:** MongoDB
- **Vector Storage Layer:** ChromaDB, FAISS
- **Tabular Analysis Engine:** DuckDB
- **Edge Microcontroller:** Arduino