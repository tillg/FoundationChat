# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FoundationChat is a SwiftUI application demonstrating Apple's Foundation Models framework for on-device AI. The project targets iOS 26.0+ and showcases chat functionality using Apple Intelligence with persistent conversation storage, streaming responses, and tool integration.

## Build and Run

This is an Xcode project (not SPM package). Dependencies are managed via Swift Package Manager within Xcode.

**Build commands:**
```bash
# Build via command line
xcodebuild -project FoundationChat.xcodeproj -scheme FoundationChat -configuration Debug

# Open in Xcode (preferred)
open FoundationChat.xcodeproj
```

**Dependencies:**
- SwiftSoup (2.8.8+) - HTML parsing for WebAnalyserTool

**Platform Requirements:**
- iOS 26.0+ / macOS 15.5+ / visionOS 26.0+
- Device with Apple Intelligence enabled (Settings > Apple Intelligence)
- Simulator not recommended - use physical device for best performance

## Codebase Architecture

**Pattern:** MVVM with SwiftUI's modern data flow

**Key Components:**

1. **Models/** - Data layer
   - `SwiftData/` - Persistent storage with @Model macro
     - `Conversation.swift` - Container for messages with cascade delete
     - `Message.swift` - Individual messages with rich attachments
   - `Generable/` - Foundation Models structured output types
     - `MessageGenerable.swift` - @Generable for AI responses
     - `WebPageMetadata.swift` - Structured web content data

2. **Views/** - UI layer
   - `ConversationsList/` - Main navigation and conversation list
   - `ConversationDetail/` - Chat interface with streaming updates
     - `Message/` - Message display components with attachment support

3. **Env/** - Business logic layer
   - `ChatEngine.swift` - Central Foundation Models integration
     - Manages LanguageModelSession lifecycle
     - Implements streaming responses
     - Handles context window management (3500 token safe limit)
     - Automatic conversation summarization

4. **Tools/** - Foundation Models tool implementations
   - `WebAnalyserTool.swift` - Extracts metadata from web pages

**Data Flow:**
```
User Input → ConversationDetailView → ChatEngine (@Environment)
→ LanguageModelSession → Tools → @Generable Output
→ SwiftData Persistence → UI Updates (Streaming)
```

**Key Design Patterns:**
- Environment-based dependency injection (ChatEngine provided at navigation level)
- SwiftData @Query for reactive data fetching
- @Observable for state management (modern replacement for ObservableObject)
- AsyncSequence streaming for real-time UI updates
- @Generable for type-safe AI responses

## Framework Overview

Apple's Foundation Models framework provides on-device large language models that power Apple Intelligence. It's available on iOS 26.0+, iPadOS 26.0+, macOS 26.0+, and visionOS 26.0+.

## Key Documentation Files

### 1. **EXAMPLES/FOUNDATION_MODELS_RULES.md**
**When to use:** Start here for understanding core framework principles
- Availability checking patterns
- Session management rules
- Performance optimization guidelines
- Safety implementation requirements
- Platform requirements and limitations

### 2. **EXAMPLES/QUICK_REFERENCE.md**
**When to use:** For quick code snippets and common patterns
- Import statements
- Basic usage patterns
- Error handling snippets
- Common configurations

### 3. **EXAMPLES/01_Basic_Usage.md**
**When to use:** For simple text generation tasks
- Creating sessions
- Basic prompting
- Handling availability states
- Error handling basics
- Multi-turn conversations

### 4. **EXAMPLES/02_Structured_Output.md**
**When to use:** When you need structured data from the model
- @Generable macro usage
- @Guide annotations
- Complex nested structures
- Optional properties
- Performance considerations

### 5. **EXAMPLES/03_Streaming_Responses.md**
**When to use:** For real-time UI updates during generation
- Basic streaming implementation
- Streaming with structured output
- Progress tracking
- Error handling in streams
- SwiftUI integration

### 6. **EXAMPLES/04_Tool_Calling.md**
**When to use:** When the model needs external data/actions
- Tool protocol implementation
- Integration with system frameworks (Contacts, Calendar, etc.)
- Stateful tools
- Multiple tool usage

### 7. **EXAMPLES/05_Performance_and_Safety.md**
**When to use:** For optimization and safety implementation
- Prewarming strategies
- Schema optimization
- Safety boundaries
- Context management
- Performance monitoring

### 8. **EXAMPLES/06_Complete_Chat_App.md**
**When to use:** For understanding the full application structure
- Complete chat implementation
- SwiftUI best practices
- State management
- UI/UX considerations

## Development Workflow

### Starting a New Feature

1. **Check FOUNDATION_MODELS_RULES.md** for constraints
2. **Reference QUICK_REFERENCE.md** for syntax
3. **Find similar examples** in the numbered example files
4. **Always check availability first**
5. **Implement with proper error handling**

### Common Tasks Reference

| Task | Primary Reference | Secondary Reference |
|------|------------------|-------------------|
| Basic text generation | 01_Basic_Usage.md | QUICK_REFERENCE.md |
| Structured data extraction | 02_Structured_Output.md | 04_Tool_Calling.md |
| Chat interface | 06_Complete_Chat_App.md | 03_Streaming_Responses.md |
| Adding tools/functions | 04_Tool_Calling.md | FOUNDATION_MODELS_RULES.md |
| Performance optimization | 05_Performance_and_Safety.md | FOUNDATION_MODELS_RULES.md |
| Safety implementation | 05_Performance_and_Safety.md | FOUNDATION_MODELS_RULES.md |

## Critical Rules (Always Remember)

1. **ALWAYS check `SystemLanguageModel.default.isAvailable`** before using
2. **NEVER assume the model is available** - provide fallback UI
3. **Handle all error cases**, especially:
   - `GenerationError.guardrailViolation`
   - `GenerationError.exceededContextWindowSize`
   - `GenerationError.unsupportedLanguageOrLocale`
4. **Use streaming for responses > 1 sentence** for better UX
5. **Prewarm sessions** when users show intent to interact
6. **Keep instructions in sessions**, not user input (security)
7. **Display errors in UI** - pass error messages to the user interface

## Code Patterns

### Standard Session Creation
```swift
@available(iOS 26.0, *)
guard SystemLanguageModel.default.isAvailable else { 
    // Show fallback UI
    return 
}

let session = LanguageModelSession(instructions: """
    You are a helpful assistant.
    Be concise and accurate.
    """)
```

### Structured Output Pattern
```swift
@Generable
struct Output {
    @Guide(description: "Clear description")
    let field: String
}

let response = try await session.respond(
    to: prompt,
    generating: Output.self
)
```

### Tool Implementation Pattern
```swift
struct MyTool: Tool {
    let name = "toolName"
    let description = "What it does"
    
    @Generable
    struct Arguments {
        let param: String
    }
    
    func call(arguments: Arguments) async throws -> ToolOutput {
        // Implementation
        return ToolOutput("result")
    }
}
```

## Debugging Tips

1. Use **Foundation Models Instrument** in Xcode
2. Monitor token counts and response times
3. Check `session.transcript` for conversation history
4. Log availability state changes
5. Test all error scenarios

## When Things Go Wrong

- **Model not available**: Check Settings > Apple Intelligence
- **Guardrail violations**: Rephrase prompts, add safety instructions
- **Context overflow**: Create new session or condense history
- **Poor performance**: Check prewarming, schema inclusion settings

## Recent Updates and Features

### Tool Integration
The project now includes the `WebAnalyserTool` that demonstrates Foundation Models tool calling:
- Extracts structured metadata from web pages (title, thumbnail, description)
- Uses SwiftSoup for HTML parsing
- Returns structured data via `@Generable` types

### Message Attachments
Messages now support rich attachments:
- `attachementTitle`: Display title for web content
- `attachementThumbnail`: Preview image URL
- `attachementDescription`: Content description

### UI Improvements
- Enhanced scrolling behavior with `.scrollPosition`
- Automatic keyboard focus on conversation load
- Error messages displayed directly in the chat UI
- Preview support for SwiftUI views

## Critical Architecture Details

### ChatEngine.swift - The Core Integration Point

This is the central class for Foundation Models integration. Key responsibilities:

1. **Session Management:**
   - Creates LanguageModelSession with system instructions
   - Configures WebAnalyserTool for web content extraction
   - Manages session lifecycle per conversation

2. **Context Window Management:**
   - Tracks token usage (4096 max, 3500 safe limit)
   - Switches between full history and summary mode automatically
   - Estimates tokens as: `(text.split(separator: " ").count * 1.3)`

3. **Streaming Implementation:**
   - `streamResponse()` - Streams MessageGenerable with real-time updates
   - `streamSummary()` - Generates conversation titles
   - Both return `AsyncThrowingStream<MessageGenerable, Error>`

4. **Tool Integration:**
   - Tools configured at session creation: `[WebAnalyserTool()]`
   - To add new tools, implement `Tool` protocol and add to array

### SwiftData Relationships

- `Conversation` → `Message` uses `@Relationship(deleteRule: .cascade)`
- Deleting a conversation automatically deletes all messages
- `@Query` in views provides automatic UI updates on data changes
- ModelContext injected via `@Environment(\.modelContext)`

### Message Attachments

Messages support rich web content via optional properties:
- `attachementTitle: String?` - Display title
- `attachementThumbnail: String?` - Image URL
- `attachementDescription: String?` - Content description

These are populated when WebAnalyserTool extracts metadata from URLs in messages.

### View Architecture

- **ConversationsListView** - Creates new ChatEngine per conversation via `.environment()`
- **ConversationDetailView** - Consumes ChatEngine with `@Environment(ChatEngine.self)`
- Each conversation destination gets its own isolated ChatEngine instance

## Development Best Practices

- When editing code, always build the project to check for errors and fix them, then rebuild
- Use previews to test UI components without running the full app
- Handle all async operations with proper error catching and UI feedback
- Test on physical devices with Apple Intelligence enabled, not simulator

## Testing with Tools

When implementing tools:
1. Define the tool conforming to the `Tool` protocol
2. Create `@Generable` structs for both Arguments and return types
3. Add the tool to the session configuration
4. Test with real URLs to verify extraction logic
5. Ensure proper error handling for network failures

Remember: This framework prioritizes privacy and runs entirely on-device. No internet connection is required for the model, but tools may access network resources when explicitly requested.