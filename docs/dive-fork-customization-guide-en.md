# Dive Project Fork and Customization Guide

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Technology Stack](#2-technology-stack)
3. [Project Structure](#3-project-structure)
4. [Core Components Analysis](#4-core-components-analysis)
5. [Data Flow](#5-data-flow)
6. [Fork and Customization Strategy](#6-fork-and-customization-strategy)
7. [Important Considerations](#7-important-considerations)
8. [Conclusion](#8-conclusion)

## 1. Project Overview

Dive is an open-source desktop application that seamlessly integrates with any LLM model supporting Function Calling capabilities. This application enables AI agents to use external tools through the MCP (Model Context Protocol).

### Key Features
- Support for various LLMs (ChatGPT, Anthropic, Ollama, OpenAI-compatible models)
- Cross-platform support (Windows, MacOS, Linux)
- MCP (Model Context Protocol) support (both stdio and SSE modes)
- Multi-language support (English, Chinese, Japanese, Spanish, etc.)
- Advanced API management (multiple API keys and model switching)
- Custom system prompts
- Automatic update mechanism

## 2. Technology Stack

### Frontend
- React
- TypeScript
- Sass/SCSS
- Jotai (state management)
- React Router
- i18next (internationalization)

### Backend
- Node.js
- Express
- LangChain
- SQLite (Better-SQLite3)
- Drizzle ORM

### Desktop Application
- Electron
- Vite

### AI/LLM Integration
- LangChain
- Model Context Protocol (MCP)
- Various LLM provider SDKs (OpenAI, Anthropic, Google, AWS, etc.)

## 3. Project Structure

The Dive project is divided into the following main components:

### 1. Electron Application (Desktop UI)
- `/electron/main/` - Electron main process
- `/electron/preload/` - Electron preload scripts
- `/src/` - React frontend application

### 2. API Services
- `/services/` - Backend service logic
- `/services/mcpServer/` - MCP server management
- `/services/models/` - LLM model management
- `/services/database/` - Database connection and schema

### 3. Tools and Utilities
- `/services/utils/` - Common utility functions
- `/services/prompt/` - Prompt template management

## 4. Core Components Analysis

### 1. MCP Server Manager (MCPServerManager)
- Manages connections to MCP servers and handles tool calls
- Supports various transport modes (stdio, SSE, WebSocket)
- Includes functionality to dynamically update server configurations

### 2. Model Manager (ModelManager)
- Manages various LLM providers and model settings
- Integrates with LangChain to provide a consistent interface
- Functionality for loading, saving, and updating model configurations

### 3. Query Processing (processQuery)
- Delivers user queries to AI and processes streaming responses
- Iterative logic for tool calls and result processing
- Error handling and abort logic

### 4. Database Management
- Chat and message storage
- Schema and migrations using Drizzle ORM

### 5. Web Application UI
- Chat interface
- Settings and configuration UI
- Multi-language support

## 5. Data Flow

1. User enters a message (`ChatInput.tsx`)
2. Message is sent to the server (`Chat/index.tsx`)
3. Server processes the message and context (`client.ts`)
4. Message is sent to LLM and responses are streamed (`processQuery.ts`)
5. If function calls occur, they are routed to the appropriate MCP server (`mcpServer/index.ts`)
6. Tool results are returned to the LLM and influence subsequent responses
7. Final response is displayed to the user and stored in the database

## 6. Fork and Customization Strategy

### 1. Branch Management Strategy
1. **Main Branches**:
   - `upstream/main`: Main branch of the original repository
   - `origin/main`: Main branch of your fork (synchronized with the original)
   
2. **Development Branches**:
   - `origin/develop`: Branch for all custom development work
   - Feature-specific branches: `feature/custom-feature-name`

### 2. Synchronizing with the Original Repository
1. Add the original repository as upstream:
   ```bash
   git remote add upstream https://github.com/OpenAgentPlatform/Dive.git
   ```

2. Fetch upstream changes:
   ```bash
   git fetch upstream
   ```

3. Merge upstream changes into the main branch:
   ```bash
   git checkout main
   git merge upstream/main
   git push origin main
   ```

4. Rebase changes to the development branch:
   ```bash
   git checkout develop
   git rebase main
   git push origin develop --force  # Note: Use force push with caution in collaborative situations
   ```

### 3. Setting up a Continuous Integration Pipeline (GitHub Actions)
You can set up a GitHub Actions workflow to automatically fetch updates from the original repository and integrate them into your forked project:

```yaml
name: Sync with upstream

on:
  schedule:
    - cron: '0 0 * * *'  # Run daily
  workflow_dispatch:  # Allow manual trigger

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          ref: main
          fetch-depth: 0

      - name: Add upstream
        run: |
          git remote add upstream https://github.com/OpenAgentPlatform/Dive.git
          git fetch upstream

      - name: Sync main branch
        run: |
          git checkout main
          git merge upstream/main
          git push origin main

      - name: Sync develop branch
        run: |
          git checkout develop
          git rebase main
          git push origin develop --force
```

### 4. Identifying Customization Areas
When forking the Dive project, it's suitable to customize the following areas:

1. **UI Themes and Styling**:
   - Modify SCSS files in the `/src/styles/` directory
   - Change colors, logos, and icons to match company branding

2. **Custom MCP Server Integration**:
   - Integrate company-specific custom tools in the `/services/mcpServer/` directory
   - Add dedicated MCP servers for internal APIs and services

3. **Authentication Mechanisms**:
   - Integrate with company SSO or authentication systems
   - Add user management and access control

4. **Enterprise Features**:
   - Team collaboration features
   - Conversation export/import
   - Administrative features

5. **Model Provider Customization**:
   - Prioritize internal or specific model providers
   - Customize default parameters and settings

### 5. Strategy for Minimal-Change Customization

1. **Using Hooks and Extension Patterns**:
   - Add new service layers without changing core logic
   - Use patterns that wrap and extend existing functions

2. **Configuration-Based Customization**:
   - Utilize configuration files instead of hardcoded changes
   - Use environment variables and setting-based conditional logic

3. **Developing a Plugin System**:
   - Design a plugin architecture for custom features
   - Extend functionality without changing the core codebase

4. **CSS Overrides**:
   - Use dedicated stylesheets that override core styles without changing them

## 7. Important Considerations

1. **License Compliance**: Dive is an open source project, so all forks and customizations must comply with the original license.

2. **Update Conflicts**: Updates from the original repository may conflict with your customizations, so minimizing changes and using extension patterns is recommended.

3. **Security Considerations**: Follow security best practices in managing API keys and credentials.

4. **Performance Optimization**: Ensure custom features do not negatively impact application performance.

## 8. Conclusion

The Dive project is a well-structured Electron application with an extensible architecture that supports various LLM providers. Its native support for MCP (Model Context Protocol) makes it easy to extend AI agent capabilities, which is particularly useful for integration with internal company tools.

When forking and customizing, implementing the suggested branch management strategy and continuous integration pipeline will allow you to continue receiving updates from the original project while expanding it to meet company-specific requirements. Adding needed functionality with minimal changes will improve maintainability and minimize update conflicts.
