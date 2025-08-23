# OpenCode Configuration Documentation

This directory contains the configuration for OpenCode, an AI-powered development assistant. This documentation will help you understand and customize the configuration.

## Directory Structure

```
/home/althaf/.config/opencode/
├── README.md           # This documentation file
├── opencode.jsonc      # Main configuration file
└── agents/
    └── pr-review.md    # PR review agent configuration
```

## Main Configuration (`opencode.jsonc`)

The main configuration file uses JSONC format (JSON with comments). Here's what each section does:

### Basic Settings

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "theme": "catpuccin",        // UI theme (catpuccin, default, etc.)
  "autoupdate": true           // Automatically update OpenCode
}
```

### Agents Configuration

Agents are specialized AI assistants for specific tasks:

```jsonc
"agent": {
  "pr-review": {
    "description": "Specialized agent for reviewing GitHub pull requests using gh CLI, focusing on code quality, security, and best practices"
  }
}
```

### MCP (Model Context Protocol) Servers

MCP servers extend OpenCode's capabilities by connecting to external services:

#### Context7 (Documentation Search)
```jsonc
"context7": {
  "type": "local",
  "command": ["npx", "-y", "@upstash/context7-mcp"],
  "enabled": true
}
```
- **Purpose**: Provides access to documentation for various libraries and frameworks
- **Status**: Currently enabled
- **Usage**: Automatically searches relevant docs when you ask questions about libraries

#### MongoDB Integration
```jsonc
"mongodb": {
  "type": "local",
  "command": [
    "npx", "-y", "mongodb-mcp-server",
    "--connectionString",
    "mongodb://${MONGODB_HOST}:${MONGODB_PORT}/?directConnection=true",
    "--readOnly"
  ],
  "enabled": false,
  "environment": {
    "MONGODB_HOST": "localhost",
    "MONGODB_PORT": "27017"
  }
}
```
- **Purpose**: Connects to MongoDB for read-only database queries
- **Status**: Currently disabled
- **To Enable**: Set `"enabled": true` and ensure MongoDB is running
- **Security**: Read-only access only

#### PostgreSQL Integration
```jsonc
"postgresql": {
  "type": "local",
  "command": [
    "npx", "@henkey/postgres-mcp-server",
    "--connection-string", 
    "postgresql://postgres:${POSTGRES_PASSWORD}@${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DB}"
  ],
  "enabled": false,
  "environment": {
    "POSTGRES_HOST": "localhost",
    "POSTGRES_PORT": "5432",
    "POSTGRES_DB": "hris",
    "POSTGRES_PASSWORD": "password"
  }
}
```
- **Purpose**: Connects to PostgreSQL for database operations
- **Status**: Currently disabled
- **To Enable**: Set `"enabled": true` and ensure PostgreSQL is running
- **Database**: Configured for "hris" database

## Specialized Agents

### PR Review Agent (`agents/pr-review.md`)

A specialized agent for GitHub pull request reviews with the following characteristics:

- **Model**: github-copilot/claude-sonnet-4
- **Temperature**: 0.1 (more deterministic responses)
- **Capabilities**: Read-only PR analysis using gh CLI
- **Restrictions**: Cannot write or edit files (review-only mode)

#### Key Features:
- **Security Focus**: Identifies vulnerabilities and sensitive data exposure
- **Code Quality**: Analyzes style, readability, and maintainability
- **Best Practices**: Checks framework conventions and design patterns
- **Performance**: Identifies inefficient algorithms and memory issues
- **Testing**: Evaluates test coverage and quality

#### Usage:
```bash
# The agent uses these gh commands:
gh pr view <number>     # Get PR details
gh pr diff <number>     # View changes  
gh pr comment <number>  # Add review comments
gh pr review <number>   # Submit formal review
```

## Customization Guide

### Enabling Database Connections

1. **For MongoDB**:
   - Set `"enabled": true` in the mongodb section
   - Ensure MongoDB is running on localhost:27017
   - Modify connection settings if needed

2. **For PostgreSQL**:
   - Set `"enabled": true` in the postgresql section
   - Ensure PostgreSQL is running on localhost:5432
   - Update database name, credentials as needed
   - **Security Note**: Consider using environment variables for passwords

### Adding New Agents

To create a new specialized agent:

1. Create a new `.md` file in the `agents/` directory
2. Use the frontmatter format from `pr-review.md` as a template
3. Add the agent reference to the main config file

### Environment Variables

The configuration supports environment variable substitution using `${VARIABLE_NAME}` syntax. This is recommended for sensitive information like passwords.

## Security Considerations

- Database connections are configured as read-only where possible
- Passwords should be stored as environment variables
- The PR review agent cannot modify code (review-only)
- MCP servers run locally for security

## Troubleshooting

- Ensure required CLI tools (`gh`, `npx`) are installed
- Check that database services are running before enabling integrations
- Review agent logs if specialized agents aren't working as expected
- Verify environment variables are set correctly for database connections