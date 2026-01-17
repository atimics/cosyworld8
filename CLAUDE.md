# CLAUDE.md - AI Assistant Guide for CosyWorld

This document provides comprehensive guidance for AI assistants working with the CosyWorld codebase.

## Project Overview

**CosyWorld** is an AI-powered Discord bot that creates and manages interactive avatars. The avatars can engage in conversations, participate in RPG-style combat, develop personalities over time, and maintain persistent memories. The project includes a web dashboard for monitoring avatar statistics.

**Version:** 0.0.8 (Lunar Sanctum Update)

## Quick Start

```bash
# Install dependencies
npm install

# Configure environment (copy .env.example to .env and fill in values)
cp .env.example .env

# Start the bot and web server
npm start
```

The bot runs on Discord and the web dashboard runs on port 3080.

## Project Structure

```
cosyworld8/
├── src/                          # Main bot application
│   ├── index.mjs                 # Entry point - Discord bot, command handlers
│   ├── config.mjs                # Configuration management
│   ├── models.config.mjs         # AI model definitions with rarity tiers
│   └── services/                 # Core business logic
│       ├── chat/                 # Conversation system
│       │   ├── ChatService.mjs       # Orchestrates conversations
│       │   ├── ConversationHandler.mjs # Narrative generation
│       │   ├── MessageHandler.mjs    # Discord message processing
│       │   ├── DecisionMaker.mjs     # AI response decisions
│       │   └── MessageProcessor.mjs  # Avatar/message filtering
│       ├── dungeon/              # RPG combat system
│       │   ├── DungeonService.mjs    # Combat mechanics
│       │   ├── DungeonLog.mjs        # Action logging
│       │   ├── AvatarManager.mjs     # Position tracking
│       │   └── tools/                # Extensible action tools
│       │       ├── BaseTool.mjs
│       │       ├── AttackTool.mjs
│       │       ├── DefendTool.mjs
│       │       ├── MoveTool.mjs
│       │       ├── MemoryTool.mjs
│       │       ├── RememberTool.mjs
│       │       └── CreationTool.mjs
│       ├── location/             # Dynamic location system
│       │   └── locationService.mjs
│       ├── utils/                # Utility functions
│       │   ├── messageChunker.mjs
│       │   ├── linkProcessor.mjs
│       │   └── rarityColors.mjs
│       ├── avatarService.mjs     # Avatar CRUD & generation
│       ├── discordService.mjs    # Discord integration
│       ├── memoryService.mjs     # Persistent memory
│       ├── openrouterService.mjs # OpenRouter AI API
│       ├── ollamaService.mjs     # Local LLM alternative
│       ├── replicateService.mjs  # Image generation
│       ├── s3imageService.mjs    # S3 image storage
│       ├── arweaveService.mjs    # Blockchain storage
│       ├── logger.mjs            # Winston logging
│       └── utils.mjs             # JSON/input utilities
├── server/                       # Express.js web server
│   ├── app.mjs                   # API routes & server setup
│   └── routes/                   # REST API endpoints
├── public/                       # Web dashboard (React)
│   ├── index.html                # Frontend entry
│   └── js/app.js                 # React leaderboard app
├── package.json                  # Dependencies (ES modules)
├── .env.example                  # Environment template
└── data_migration.js             # Database migration utility
```

## Key Technologies

| Technology | Purpose | Version |
|------------|---------|---------|
| Node.js | Runtime (ES Modules) | v18+ |
| Discord.js | Discord bot framework | v14.16.3 |
| Express.js | Web server | v4.21.1 |
| MongoDB | Document database | v6.11.0 |
| OpenRouter | LLM API provider | - |
| Replicate | Image generation | v1.0.1 |
| React | Dashboard UI | v18 (CDN) |
| Tailwind CSS | Styling | CDN |
| Winston | Logging | v3.15.0 |

## Architecture Patterns

### Service-Oriented Architecture

All business logic resides in `/src/services/`. Each service is responsible for a specific domain:

- **ChatService** - Orchestrates avatar conversations, manages response rates
- **DungeonService** - Handles RPG combat mechanics and tools
- **AvatarService** - Avatar CRUD, image generation, breeding
- **MemoryService** - Persistent memory storage/retrieval
- **LocationService** - Dynamic channel/location creation
- **DiscordService** - Discord API wrapper, webhooks, permissions

### Tool Pattern (Dungeon System)

Combat actions extend `BaseTool.mjs`:

```javascript
// src/services/dungeon/tools/ExampleTool.mjs
import BaseTool from './BaseTool.mjs';

export default class ExampleTool extends BaseTool {
  constructor(dungeonService) {
    super(dungeonService);
    this.name = 'example';
  }

  async execute(actor, target, params) {
    // Implementation
  }
}
```

### Async/Await Pattern

All async operations use async/await with try-catch error handling:

```javascript
async function example() {
  try {
    const result = await someAsyncOperation();
    return result;
  } catch (error) {
    logger.error('Operation failed:', error);
    throw error;
  }
}
```

## Database Schema (MongoDB)

### Collections

| Collection | Purpose | Key Fields |
|------------|---------|------------|
| `avatars` | Core avatar data | name, emoji, personality, description, imageUrl, model, dynamicPersonality, channelId, parents[], status, lives |
| `messages` | Discord message history | messageId, channelId, authorId, authorUsername, content, timestamp |
| `memories` | Avatar memories | avatarId, memory, timestamp |
| `narratives` | Generated narratives | avatarId, content, timestamp |
| `dungeon_stats` | Combat statistics | avatarId, hp, attack, defense |
| `dungeon_log` | Combat history | actor, target, action, timestamp |
| `dungeon_positions` | Avatar locations | avatarId, locationId, areaId |
| `locations` | World locations | name, description, imageUrl, createdAt |
| `tribes` | Avatar groupings | emoji, members[], count |

### Default Values

- **HP**: 100
- **Attack**: 10
- **Defense**: 5
- **Lives**: 3
- **Response Rate**: 5% (ambient)
- **Narrative Cooldown**: 6 hours
- **Breeding Cooldown**: 24 hours

## Environment Variables

Required in `.env`:

```env
# Discord
DISCORD_BOT_TOKEN="..."

# MongoDB
MONGO_URI="mongodb://127.0.0.1:27017"
MONGO_DB_NAME="cosyworld"

# AI (OpenRouter)
OPENROUTER_API_TOKEN="..."
OPENROUTER_MODEL="openai/gpt-4o"

# Image Generation
REPLICATE_API_TOKEN="..."
REPLICATE_MODEL="immanencer/mirquo:..."

# Storage
S3_API_ENDPOINT="..."
S3_API_KEY="..."
CLOUDFRONT_DOMAIN="..."

# Optional
OLLAMA_MODEL="..."  # Local LLM alternative
```

## Bot Commands

Commands are processed in `src/index.mjs`:

| Command | Description |
|---------|-------------|
| `!summon [name/description]` | Create or summon an avatar |
| `!breed [avatar1] [avatar2]` | Breed two avatars (24h cooldown) |
| `!attack [avatar]` | Initiate combat |

## API Endpoints

Web server runs on port 3080 (`server/app.mjs`):

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/api/health` | Health check |
| GET | `/api/leaderboard` | Avatar rankings |
| GET | `/api/avatars/:id` | Avatar details with ancestry |
| GET | `/api/avatars/search` | Search avatars |
| GET | `/api/avatar/:id/narratives` | Avatar narratives |
| GET | `/api/avatar/:id/memories` | Avatar memories |
| GET | `/api/avatar/:id/dungeon-actions` | Combat history |
| GET | `/api/dungeon/log` | Recent combat log |
| GET | `/api/tribes` | Avatar groupings |

## AI Model Rarity System

Models are assigned based on rarity tiers (`src/models.config.mjs`):

| Tier | Probability | Examples |
|------|-------------|----------|
| Legendary (5%) | Roll 20 | GPT-4o, Claude 3 Opus/Sonnet, LLaMA 405B, Grok |
| Rare (10%) | Roll 18-19 | EVA Qwen 72B, Lumimaid 70B, Nemotron 70B |
| Uncommon (25%) | Roll 13-17 | Claude 3.5 Haiku, Mistral Large, Mythalion |
| Common (60%) | Roll 1-12 | LLaMA 3.2 1B/3B, GeminiFlash, MythoMax |

## Key Implementation Details

### Response Logic

- **Ambient Response Rate**: 5% chance to respond to channel messages
- **Mention Detection**: Higher response rate when avatar is mentioned
- **Cooldowns**: 5-minute window for duplicate prevention
- **Reflection Interval**: 1 hour between reflection/narrative generation

### Combat Calculation

```javascript
damage = Math.max(0, attacker.attack - target.defense);
target.hp = Math.max(0, target.hp - damage);
if (target.hp <= 0) {
  target.lives--;
  if (target.lives > 0) target.hp = 100; // Respawn
}
```

### Message Processing Flow

1. Parse messages for `!` commands
2. Handle commands (max 2 per message)
3. Save non-command messages to database
4. Trigger channel processing for human messages
5. MessageHandler checks for avatar mentions
6. DecisionMaker determines if avatar should respond
7. ConversationHandler generates response with context

### Webhook Caching

Discord webhooks are cached per channel for performance:

```javascript
// In discordService.mjs
const webhookCache = new Map();
```

## Code Conventions

### File Naming

- ES Modules: `.mjs` extension
- PascalCase for service files: `ChatService.mjs`
- camelCase for utility files: `messageChunker.mjs`

### Imports

```javascript
// External dependencies first
import { Client, GatewayIntentBits } from 'discord.js';
import { MongoClient } from 'mongodb';

// Internal services second
import { logger } from './services/logger.mjs';
import AvatarService from './services/avatarService.mjs';
```

### Error Handling

Always wrap async operations:

```javascript
try {
  const result = await operation();
} catch (error) {
  logger.error('Context-specific message:', error);
  // Graceful degradation or re-throw
}
```

### Logging

Use Winston logger (`src/services/logger.mjs`):

```javascript
import { logger } from './services/logger.mjs';

logger.info('Informational message');
logger.error('Error message', error);
logger.debug('Debug message');
```

## Testing

**Note:** No test suite is currently configured (`npm test` exits with error).

When adding tests, consider:
- Unit tests for services
- Integration tests for API endpoints
- Mock Discord.js client for bot tests

## Common Tasks

### Adding a New Dungeon Tool

1. Create file in `src/services/dungeon/tools/`
2. Extend `BaseTool.mjs`
3. Register in `DungeonService.mjs`

### Adding an API Endpoint

1. Add route in `server/app.mjs`
2. Use async handler with try-catch
3. Return JSON responses

### Adding a Bot Command

1. Add command handler in `src/index.mjs`
2. Parse in the message event handler
3. Follow existing `!command` pattern

### Modifying Avatar Generation

Key files:
- `src/services/avatarService.mjs` - Core logic
- `src/services/replicateService.mjs` - Image generation
- `src/models.config.mjs` - Model rarity configuration

## Performance Considerations

- **Message Chunking**: Discord has 2000 char limit
- **Parallel Fetching**: Use `Promise.all()` for independent operations
- **Webhook Caching**: Reuse webhooks per channel
- **Activity Tracking**: 30-minute timeout for inactive avatars
- **Memory Limits**: Default 10 recent memories + 3 narratives

## Security Notes

- Environment variables for all secrets
- Input sanitization (alphanumeric + emoji only)
- Permission checking before Discord actions
- `.gitignore` for sensitive files

## Debugging Tips

1. Check Winston logs for error traces
2. Verify MongoDB connection string
3. Ensure Discord bot has required permissions
4. Check OpenRouter API key and model availability
5. Verify S3/Cloudfront configuration for images

## File Reference

| File | Lines | Purpose |
|------|-------|---------|
| `src/index.mjs` | ~620 | Main entry, command handlers |
| `server/app.mjs` | ~560 | Web server, API routes |
| `src/services/chat/ChatService.mjs` | ~360 | Chat orchestration |
| `src/services/avatarService.mjs` | ~250 | Avatar management |
| `src/services/dungeon/DungeonService.mjs` | ~250 | Combat system |
| `src/services/location/locationService.mjs` | ~375 | Location management |
| `public/js/app.js` | ~500 | React dashboard |
