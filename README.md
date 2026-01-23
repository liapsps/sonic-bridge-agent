# Sonic Bridge Agent 🎵

> A conversational recommendation system that bridges user profiles with their perfect soundtrack.

**Sonic Bridge** is a Proof of Concept (PoC) for an Agentic Recommendation System. It leverages **LangGraph** to manage conversational state and **LangChain** to extract and persist structured data from natural language.

## 🎯 Objective

This project demonstrates a stateful AI agent that:

1. **Profiles:** Extracts specific entities (`name`, `age`, `gender`) from conversation.
2. **Persists:** Stores extracted data in a temporary state database to ensure session durability.
3. **Validates:** Uses a "Human-in-the-loop" workflow for missing profile data.
4. **Analyzes:** Translates natural language intent into technical Spotify API parameters.

## 🏗 Architecture

The system uses a state machine where the **Profile Storage** acts as the single source of truth.

```mermaid
graph TD
    %% Style Definitions
    classDef llm fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef logic fill:#fff9c4,stroke:#fbc02d,stroke-width:2px;
    classDef tool fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    classDef user fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px;
    classDef storage fill:#ffe0b2,stroke:#e65100,stroke-width:2px;

    Start((Start)) --> UserInput[User Input]
    UserInput --> Extractor[Node: Profile Extractor]
    
    %% Storage Node: Persisting entities
    Extractor --> DB[(Node: Profile Storage)]
    
    %% Logic reads from DB
    DB --> CheckState{JSON Complete?}
    
    %% Loop (Human-in-the-loop)
    CheckState -- No --> Asker[Node: Question Generator]
    Asker --> UserInput
    
    %% Recommendation Flow: Intent Analyzer uses DB context
    CheckState -- Yes --> IntentAnalyzer[Node: Intent Analyzer]
    DB -.-> IntentAnalyzer
    
    IntentAnalyzer --> SpotifyTool[Tool: Spotify API Search/Recs]
    SpotifyTool --> FinalResponse[Node: Final Response]
    
    FinalResponse --> End((End))

    %% Assign classes
    class Extractor,Asker,IntentAnalyzer,FinalResponse llm;
    class CheckState logic;
    class SpotifyTool tool;
    class UserInput user;
    class DB storage;

```

### Flow Breakdown

1. **Profile Extractor:** Populates the required JSON schema (`name`, `age`, `gender`).
2. **Profile Storage:** Persists the JSON state (SQLite/Redis). This ensures the agent doesn't "forget" information during the loop.
3. **Question Generator (Loop):** Generates context-aware questions for missing data.
4. **Intent Analyzer:** Reads the consolidated profile and maps user "vibes" to Spotify parameters (e.g., `valence`, `energy`).
5. **Spotify Tool:** Queries the API using the translated technical parameters.

## 🛠 Tech Stack

* **Orchestration:** LangGraph & LangChain
* **External API:** Spotify Web API (via `spotipy`)
* **Persistence:** SQLite / Local State JSON (for session management)
* **Validation:** Pydantic
