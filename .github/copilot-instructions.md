# Copilot Instructions for AI Chat Demo

## Architecture Overview

This is a **RAG (Retrieval-Augmented Generation) chat application** built with **Blazor Server** (.NET 9) that enables AI conversations over custom PDF documents. The architecture follows a clear separation:

- **Frontend**: Blazor Server components with streaming chat UI (`Components/Pages/Chat/`)
- **AI Integration**: Microsoft Extensions AI with OpenAI via GitHub Models
- **Vector Storage**: SQLite with vector extensions for semantic search
- **Document Processing**: PDF ingestion with chunking and embedding generation

## Key Components & Data Flow

1. **Document Ingestion** (`Services/Ingestion/`): PDF files from `wwwroot/Data/` → text extraction → chunking → embeddings → SQLite vector store
2. **Chat Interface** (`Chat.razor`): User query → semantic search → AI function calling → streamed response with citations
3. **Vector Search** (`SemanticSearch.cs`): Embedding-based similarity search with optional document filtering

## Critical Developer Workflows

### Setup & Configuration
```bash
# Required: Set GitHub Models token
dotnet user-secrets set GitHubModels:Token YOUR-TOKEN

# Run with PDF ingestion on startup
dotnet run
```

### Key Service Registration Pattern
The app uses **builder pattern** in `Program.cs` with specific AI service extensions:
- `AddChatClient(chatClient).UseFunctionInvocation().UseLogging()`
- `AddSqliteCollection<string, IngestedChunk>()` for vector collections
- Automatic PDF ingestion via `DataIngestor.IngestDataAsync()` on startup

### Vector Store Schema
- **IngestedChunk**: Text chunks with 1536-dimension embeddings (OpenAI text-embedding-3-small)
- **IngestedDocument**: Document metadata tracking for incremental updates
- Collections created/updated automatically via `EnsureCollectionExistsAsync()`

## Project-Specific Patterns

### AI Function Calling
The chat uses **function calling** for semantic search:
```csharp
[Description("Searches for information using a phrase or keyword")]
private async Task<IEnumerable<string>> SearchAsync(
    [Description("The phrase to search for.")] string searchPhrase,
    [Description("Optional filename filter")] string? filenameFilter = null)
```

### Citation Format
Responses must include XML citations:
```xml
<citation filename='string' page_number='number'>exact quote here</citation>
```

### Streaming Response Pattern
Uses `GetStreamingResponseAsync()` with real-time UI updates via `ChatMessageItem.NotifyChanged()`

## File Organization Conventions

- **Services/**: Core business logic (semantic search, ingestion)
- **Services/Ingestion/**: Document processing pipeline with `IIngestionSource` abstraction
- **Components/Pages/Chat/**: Chat UI components (modular design)
- **wwwroot/Data/**: PDF documents to ingest (replaced on startup)
- **wwwroot/lib/**: Client-side libraries (PDF.js, Marked, DOMPurify, Tailwind)

## External Dependencies

- **GitHub Models**: AI provider (requires token in user secrets)
- **PdfPig**: PDF text extraction
- **SQLite + vector extensions**: Local vector database
- **Microsoft.Extensions.AI**: Unified AI abstractions
- **Semantic Kernel**: Vector database connectors

## Development Notes

- Vector store file: `vector-store.db` in app base directory
- System prompt defines AI behavior and citation requirements
- PDF ingestion runs automatically on app startup
- Blazor Server mode enables real-time streaming without client-side JS
- User secrets required for GitHub Models token (not appsettings.json)
