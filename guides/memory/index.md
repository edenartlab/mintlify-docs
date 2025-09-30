# Eden Memory System

Eden's memory system enables AI agents to learn and evolve through conversations, creating persistent context that makes interactions more personalized and effective over time. The system operates automatically in the background, extracting and organizing important information from conversations.

## Overview

The memory system provides two complementary approaches to context management:

- **User Memory (Personal)**: Stores personalized preferences and instructions specific to each user's relationship with your agent
- **Collective Memory (Shared)**: Enables agents to learn from all interactions, building shared knowledge across your entire community

## Key Features

### Automatic Memory Formation
- Memories are extracted automatically during conversations
- No manual intervention required
- Smart triggers based on conversation length and content

### Intelligent Organization
- **Episodes**: Conversation summaries for session continuity
- **Directives**: Persistent user preferences and behavioral instructions
- **Facts**: Atomic, verifiable information in a shared knowledge base
- **Suggestions**: New ideas and proposals for evolving collective context

### Customizable Behavior
- Memory Instructions guide what information to capture
- Flexible consolidation thresholds
- Per-shard configuration options
- Active/paused state control

## How It Works

The memory system automatically monitors conversations and extracts relevant information when thresholds are met:

1. **Extraction**: Identifies important information from recent messages
2. **Storage**: Saves memories with full traceability and metadata
3. **Consolidation**: Periodically merges and refines memories
4. **Context Assembly**: Provides complete memory context for agent responses

## Use Cases

### Personal AI Assistants
Create agents that remember user preferences, working styles, and individual needs across all conversations.

### Collaborative Projects
Build shared knowledge bases where agents learn from all team members and maintain project context.

### Community Agents
Deploy agents that evolve based on collective interactions, becoming more knowledgeable over time.

### Creative Workflows
Develop agents that remember artistic preferences, project requirements, and creative direction.

## Getting Started

To start using the memory system:

1. **Enable memory** for your agent in the agent settings
2. **Configure Memory Instructions** to guide what information should be captured
3. **Start conversations** and let the system learn automatically
4. **Review and refine** memories through the management interface

## Documentation

- **[Complete Guide](guide.md)**: Detailed documentation on architecture, configuration, and best practices
- **[API Reference](guide.md#api-endpoints)**: Integration endpoints and usage examples
- **[Best Practices](guide.md#best-practices)**: Tips for optimizing memory performance
- **[Troubleshooting](guide.md#troubleshooting)**: Common issues and solutions

## Benefits

- **Personalized Experiences**: Each user gets tailored interactions based on their preferences
- **Continuous Learning**: Agents improve and adapt over time
- **Efficient Context Management**: Automatic summarization and organization
- **Full Traceability**: Complete audit trails for all memory formation
- **Flexible Control**: Customize behavior through Memory Instructions
- **No Manual Effort**: Set it up once and let it work automatically

---

_The memory system is available for all Eden agents. Collective Memory requires Pro+ tier or Preview access._