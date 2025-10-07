# Eden Memory System Documentation

## Overview

Eden's memory system enables your agents to learn and evolve through conversations, creating persistent context that makes interactions more personalized and effective over time. The system operates automatically in the background, extracting and organizing important information from conversations.

## Memory Architecture

Eden uses two complementary memory systems:

### User Memory (Personal)
User Memory stores personalized preferences and instructions specific to each user's relationship with your agent. This memory is private to each user and only affects their own interactions.

### Collective Memory (Shared)
Collective Memory is shared across all users interacting with your agent. It allows your agent to learn and evolve based on collective group interactions, making it ideal for community-driven projects, collaborative work, and agents that need to maintain shared context.

---

## Memory Components

The memory system extracts and organizes four types of memories:

### 1. Episodes
- **What**: Summaries of conversation segments that capture key events, decisions, and context
- **Purpose**: Provide conversational continuity within a session
- **Scope**: Session-specific
- **Max stored**: 8 most recent episodes per session
- **Example**: "User requested a story about a clockmaker and provided feedback that they preferred shorter paragraphs"

### 2. Directives (User Memory)
- **What**: Persistent user preferences and behavioral instructions
- **Purpose**: Customize agent behavior for individual users
- **Scope**: User-specific, persists across all sessions
- **Consolidation**: Automatically merged when 5+ directives accumulate (production) or 3+ (development)
- **Example**: "Always ask Jack for permission before generating videos"

### 3. Facts (Collective Memory)
- **What**: Atomic, verifiable, unchanging information relevant to the collective memory context
- **Purpose**: Build a permanent knowledge base
- **Scope**: Shared across all users
- **Storage**: FIFO queue with max 50 facts per memory shard (production)
- **Example**: "The project deadline is May 1st (source: Alice)" or "Maximum budget is $1000 (source: Bob)"

### 4. Suggestions (Collective Memory)
- **What**: New ideas, proposals, or context updates for the collective memory
- **Purpose**: Allow the collective memory to evolve based on community input
- **Scope**: Shared across all users
- **Consolidation**: Automatically integrated when 16+ suggestions accumulate (production)
- **Example**: "Bob proposed adding a feature that allows users to export data in CSV format"

---

## Memory Formation Flow

### Automatic Trigger
Memories are automatically formed when either:
- **Message threshold**: 45+ messages since last memory formation (production)
- **Token threshold**: 1000+ tokens since last memory formation (production)

### Extraction Process

**Step 1: Episode & Directive Extraction**
The system analyzes recent conversation messages and extracts:
- Exactly one episode summarizing what happened
- 0-3 directives (only if persistent user preferences are identified)

**Step 2: Collective Memory Extraction** (if active memory shards exist)
For each active collective memory shard:
- Extract 0-2 facts relevant to the shard's context
- Extract 0-5 suggestions relevant to the shard's context
- Each extracted memory is tagged with its originating shard

**Step 3: Memory Storage**
All extracted memories are saved to the database with full traceability:
- Source session ID
- Source message IDs
- Related user IDs
- Timestamp
- For collective memories: associated shard ID

### Consolidation Process

**User Memory Consolidation**
When unabsorbed directives reach the threshold (5+ in production):
1. Load all unabsorbed directive memories
2. Use LLM to merge them with existing consolidated user memory
3. Remove redundancies and contradictions (newer overrides older)
4. Update the fully formed user memory
5. Clear unabsorbed directives

**Collective Memory Consolidation**
When unabsorbed suggestions reach the threshold (16+ in production):
1. Load all unabsorbed suggestions and current facts
2. Use LLM to integrate suggestions into the consolidated memory blob
3. Maintain structure and preserve existing information
4. Update version number
5. Generate fully formed memory shard

---

## Customizing Memory Behavior

### Memory Instructions (Shard Extraction Prompt)

The **Memory Instructions** field is the most important setting for collective memory. This prompt tells your agent:
- What information to pay attention to
- What types of facts should be remembered
- What kinds of suggestions are relevant
- The overall purpose and context of this memory shard

**Example Memory Instructions:**
```
This collective memory is for tracking the development of our community art project "Digital Dreams".

Important to remember:
- Artist contributions and their specialties
- Technical requirements and constraints
- Agreed-upon artistic direction and themes
- Deadlines and milestone dates
- Budget allocations
- Community feedback and preferences

Focus on facts that won't change (like budget limits, deadlines) and suggestions that help guide the project's evolution (like new theme ideas, workflow improvements).
```

**Best Practices:**
1. **Be specific** about the context and purpose of the memory shard
2. **Clearly define** what qualifies as a "fact" vs a "suggestion"
3. **Iterate frequently** - the quality of your Memory Instructions directly impacts memory quality
4. **Keep it focused** - narrow scopes work better than trying to remember everything
5. **Include examples** of the types of information you want captured

### Memory Extraction Prompts (System-Level)

While users primarily interact with Memory Instructions, the system uses several internal prompts:

**Regular Memory Extraction Prompt** (episodes and directives)
- Extracts conversation summaries and persistent user preferences
- Emphasizes "what happened" and "why it matters"
- Strictly filters directives (most conversations have none)
- Fixed template, not user-customizable

**Agent Memory Extraction Prompt** (facts and suggestions)
- Uses your Memory Instructions as context
- Extracts facts and suggestions relevant to the shard
- Fixed template, not user-customizable

**User Memory Consolidation Prompt**
- Merges new directives with existing user memory
- Removes redundancies while preserving important details
- Fixed template, not user-customizable

**Agent Memory Consolidation Prompt**
- Uses your Memory Instructions as context
- Integrates suggestions into collective memory
- Maintains structure and tracks version changes
- Fixed template, not user-customizable

---

## User Interface Guide

### Accessing Memory Settings

Navigate to your agent's memory settings through the agent management interface. You'll see two tabs:

**User Memory Tab**
- View your personal memory with the agent
- Edit consolidated user memory content
- View and manage recent directives waiting for consolidation
- Toggle user memory on/off (agent owners only)

**Collective Memory Tab** (Pro+ tier or Preview access required)
- View and manage collective memory shards
- Edit Memory Instructions
- View consolidated memory content
- View and manage Facts
- View and manage Recent Memory Context (unabsorbed suggestions)
- Toggle memory shard active/paused state

### Managing User Memory

**Viewing Your Memory**
- The main text area shows your consolidated user memory
- Below that, "Recent Memory Context" shows new directives awaiting consolidation
- These recent directives will be automatically merged into your main memory

**Editing User Memory**
1. Click the edit button (appears on hover)
2. Modify the memory text in the modal
3. Confirm your changes
4. ⚠️ **Warning**: Manual edits replace the consolidated memory completely

**Managing Recent Directives**
- Click edit to modify a directive before consolidation
- Click delete to remove unwanted directives
- These will automatically merge into your main memory when the threshold is reached

### Managing Collective Memory

**Editing Memory Instructions**
1. Click the edit button in the Memory Instructions section
2. Update the prompt to better describe what should be remembered
3. Save your changes
4. This affects future memory extractions immediately

**Viewing Memory Content**
- The Memory section shows the main consolidated collective memory
- This evolves automatically as suggestions are consolidated
- Version number indicates how many consolidation cycles have occurred

**Managing Facts**
- Facts are shown with their age (e.g., "5 days old")
- Click edit to modify a fact
- Click delete to remove irrelevant facts
- Facts follow FIFO (first in, first out) - oldest facts are automatically removed when the limit is reached

**Managing Recent Memory Context** (Suggestions)
- Shows new suggestions awaiting consolidation
- Tagged with "Will be integrated soon"
- Click edit to refine suggestions before consolidation
- Click delete to remove off-topic suggestions
- These automatically consolidate when threshold is reached

**Activating/Pausing Memory**
- Use the toggle switch to activate or pause a memory shard
- Paused shards don't extract new memories but retain existing content
- Useful for temporary memory contexts or testing

---

## Memory Formation Timing

**When Memories Form:**
Memories form automatically during conversation when either:
- 45+ messages have been exchanged (production) OR
- 1000+ tokens worth of conversation has occurred (production)

Minimum requirement: At least 4 messages must have occurred since the last memory formation.

**Message Weighting:**
Different message types have different importance for triggering memory formation:
- User messages: 1.0x weight (highest priority)
- Tool results: 0.5x weight
- Agent messages: 0.2x weight (memories should come from users, not agents)

This ensures memory formation is triggered by meaningful user input rather than verbose agent responses.

---

## Memory Context Assembly

When your agent responds to a message, it assembles a complete memory context:

**For User Memory:**
1. Load the fully formed user memory (consolidated + recent directives)
2. Include username and memory age metadata
3. Format as XML structure

**For Collective Memory:**
1. Load all active memory shards
2. For each shard, include:
   - Shard name and Memory Instructions
   - Current facts with age information
   - Consolidated memory content
   - Recent suggestions awaiting consolidation
3. Format as hierarchical XML

**For Episode Memory:**
1. Load up to 8 most recent episodes from the current session
2. Order chronologically (oldest to newest)
3. Format as conversation context

**Final Memory XML Structure:**
```xml
<memory_contents description="Your complete memory context for this conversation">

  <collective_memory description="Shared memory across all your conversations">
    <memory_shard name="project_name">
      ## Shard facts:
      [Facts with age]

      ## Current consolidated shard memory:
      [Consolidated memory content]

      ## Recent shard suggestions:
      [Unabsorbed suggestions]
    </memory_shard>
  </collective_memory>

  <user_memory description="Memory and context specific to this user">
    [Consolidated user memory]

    ## Recent user directives:
    [Unabsorbed directives]
  </user_memory>

  <current_conversation_context description="Recent exchanges from this conversation">
    - [Episode 1]
    - [Episode 2]
    - [Episode N]
  </current_conversation_context>

</memory_contents>
```

This complete memory context is injected into the agent's prompt for every response.

---

## Advanced Topics

### Memory Shard Configuration

Each collective memory shard can be customized with:

**max_facts_per_shard** (default: 50 in production)
- Maximum number of facts to store in FIFO queue
- When limit is reached, oldest facts are removed
- Can be customized per shard

**max_agent_memories_before_consolidation** (default: 16 in production)
- Number of suggestions before automatic consolidation
- Higher values mean less frequent but larger consolidations
- Can be customized per shard

### Memory Caching & Performance

**Session-Level Caching:**
- Assembled memory context is cached for 5 minutes
- Episode memories are cached until memory formation occurs
- Reduces database queries for repeated requests

**Freshness Checks:**
- System checks if other sessions have updated memories
- If memories are updated elsewhere, cache is refreshed
- Ensures consistency across concurrent sessions

### Token Limits

**Maximum Memory Sizes:**
- Episode: 50 words target
- Directive: 25 words target
- Suggestion: 35 words target
- Fact: 30 words target
- User Memory Blob: 250 words target
- Agent Memory Blob: 750 words target

These are targets, not hard limits. The LLM is instructed to stay within these bounds during extraction and consolidation.

### Dynamic Memory Limits

The system uses logarithmic scaling to adjust memory extraction limits based on conversation length:

**Base Formula:**
```python
multiplier = log_base(total_chars / base_chars, log_power) + 1
multiplier = clamp(multiplier, 0.5, 2.0)
```

This means:
- Short conversations (< 3000 chars): Extract fewer memories
- Medium conversations (~3000 chars): Use standard limits
- Long conversations (> 3000 chars): Extract more memories (up to 2x)

This prevents over-extraction from short conversations and under-extraction from long ones.

---

## Best Practices

### For User Memory:
1. **Be clear and specific** when giving the agent instructions
2. **Phrase directives as rules** rather than requests (e.g., "Always X" instead of "Could you X?")
3. **Review recent directives** periodically to catch unwanted extractions
4. **Keep consolidated memory focused** on truly persistent preferences

### For Collective Memory:
1. **Start simple** - Write initial Memory Instructions that are clear and narrow in scope
2. **Iterate frequently** - Refine Memory Instructions based on what gets extracted
3. **Remove bad facts** - Clean up irrelevant facts regularly to maintain quality
4. **Review suggestions before consolidation** - Edit or delete suggestions that don't align with your goals
5. **Monitor memory version** - Track how often consolidation occurs and adjust thresholds if needed
6. **Use specific language** in Memory Instructions about what qualifies as a fact vs suggestion
7. **Provide examples** in Memory Instructions of the types of information you want

### For Both:
1. **Let the system work** - Give it time to learn from multiple conversations
2. **Trust the consolidation process** - Manual edits should be rare
3. **Use descriptive names** for memory shards
4. **Pause unused shards** rather than deleting them
5. **Monitor memory growth** - If memories grow too large, refine extraction criteria

---

## Troubleshooting

**Problem: Too many irrelevant memories are being extracted**
- Solution: Make Memory Instructions more specific about what's relevant
- Solution: Review and delete bad memories before consolidation
- Solution: Reduce max_items in memory type configuration (requires system access)

**Problem: Important information isn't being captured**
- Solution: Explicitly mention in Memory Instructions what types of information matter
- Solution: Ensure users are providing information clearly in conversations
- Solution: Check that the memory shard is active (not paused)

**Problem: Consolidated memory is becoming too verbose**
- Solution: Manually edit to reduce length
- Solution: Adjust word count targets in Memory Instructions
- Solution: More frequently review and remove outdated information

**Problem: Memory consolidation isn't happening**
- Solution: Check that enough suggestions/directives have accumulated
- Solution: Verify the memory shard is active
- Solution: Check system logs for consolidation errors

**Problem: Memories from different topics are mixing**
- Solution: Create separate memory shards for different topics
- Solution: Make Memory Instructions more focused and specific
- Solution: Use descriptive shard names that clearly indicate purpose

---

## Technical Details

### Models Used
- **Fast model** (gpt-4o-mini): Episode, directive, and user memory consolidation
- **Slow model** (gpt-4o): Fact, suggestion extraction, and collective memory consolidation

### Structured Output
Memory extraction uses structured JSON output with validation:
- Max length constraints enforced per memory type
- Dynamic limits based on conversation length
- Pydantic models ensure type safety

### Traceability
Every memory includes:
- Source session ID
- Source message IDs (which messages contributed to this memory)
- Related user IDs (who was involved)
- Timestamp (when it was created)
- For collective memories: Shard ID (which memory shard it belongs to)

This enables full audit trails and debugging.

### Race Condition Prevention
The system uses atomic MongoDB operations to prevent duplicate memory formation:
1. Check if memories should form
2. **Immediately claim** messages by updating last_memory_message_id
3. Process memory extraction and storage
4. Reset counters

This ensures multiple concurrent requests don't form duplicate memories from the same messages.

---

## API Endpoints

Users interact with memories through these API endpoints:

### User Memory
- `GET /api/agents/{agent_id}/memory` - Get user memory
- `POST /api/agents/{agent_id}/memory` - Update user memory
- `PATCH /api/agents/{agent_id}/user-memory-enabled` - Toggle user memory on/off
- `DELETE /api/agents/{agent_id}/memory/{directive_id}` - Delete a directive

### Collective Memory
- `GET /api/agents/{agent_id}/collective-memory` - Get all memory shards
- `POST /api/agents/{agent_id}/collective-memory` - Create a new memory shard
- `PATCH /api/agents/{agent_id}/collective-memory/{shard_id}` - Update shard settings
- `DELETE /api/agents/{agent_id}/collective-memory/{shard_id}/memory/{memory_id}` - Delete a fact or suggestion

### Memory Sessions
- `PATCH /api/memory-sessions/{memory_id}` - Edit individual memory content

---

## Future Enhancements

The memory system is actively evolving. Potential future improvements include:

- **Multi-agent sessions**: Better support for conversations with multiple agents
- **Memory search**: Find specific memories by content or time range
- **Memory analytics**: Track memory growth and usage patterns
- **Custom consolidation schedules**: Per-shard consolidation timing
- **Memory export**: Download memory data for external analysis
- **Memory templates**: Pre-built Memory Instructions for common use cases
- **Cross-shard references**: Link related information across memory shards
- **Memory permissions**: Fine-grained access control for collaborative agents

---

## Summary

Eden's memory system provides powerful, automatic context management for your agents:

- **User Memory** creates personalized experiences for each user
- **Collective Memory** enables agents to learn from all interactions
- **Automatic extraction** captures important information without manual effort
- **Smart consolidation** keeps memories concise and relevant
- **Memory Instructions** give you precise control over what's remembered
- **Full traceability** ensures you can audit and understand memory formation

By understanding and leveraging these features, you can create agents that truly learn and evolve over time, providing increasingly valuable and personalized interactions for your users.