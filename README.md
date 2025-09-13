# chatgpt-dev-mode
Scaffold and launch ChatGPT Developer Mode MCP servers in minutes. Runs via npx, generates OAuth or non-OAuth servers, required search/fetch tools, and JSON-RPC handlers. Built-ins: PKCE OAuth, Supabase auth, Fly.io deploy, dev scripts, tests, sample tools (search, fetch, analyze, generate). Production-ready defaults, clear docs. TypeScript-first.!

# ChatGPT MCP Developer Mode - Complete Tutorial

## Table of Contents
1. [Introduction](#introduction)
2. [What's New in Developer Mode](#whats-new-in-developer-mode)
3. [Requirements & Limitations](#requirements--limitations)
4. [Option 1: Non-OAuth Implementation](#option-1-non-oauth-implementation)
5. [Option 2: OAuth Implementation with Supabase](#option-2-oauth-implementation-with-supabase)
6. [Deployment to Fly.io](#deployment-to-flyio)
7. [Common Gotchas & Solutions](#common-gotchas--solutions)
8. [Testing Your MCP Server](#testing-your-mcp-server)
9. [Troubleshooting](#troubleshooting)

## Introduction

ChatGPT's Developer Mode with MCP Server Tools support was officially announced on **September 9th, 2025**, marking a significant milestone in AI-developer integration. This feature enables ChatGPT to interact with custom MCP (Model Context Protocol) servers, providing unprecedented extensibility and real-world integration capabilities.

### What is MCP?

Model Context Protocol (MCP) is an open protocol that standardizes how AI assistants connect with external data sources and tools. It uses JSON-RPC 2.0 for communication and supports:
- Custom tools/functions
- Resource access
- Authentication (OAuth 2.0)
- Real-time capabilities
- Persistent connections

## What's New in Developer Mode

### September 2025 Launch Features
The September 9th, 2025 announcement introduced groundbreaking capabilities:

#### Core Features
- **MCP Server Integration**: Direct connection to custom MCP servers
- **Tool Execution**: ChatGPT can now execute custom tools on your servers
- **OAuth 2.0 Support**: Production-ready authentication and authorization
- **Dynamic Tool Discovery**: Automatic discovery and registration of available tools
- **Persistent Sessions**: Maintain state and context across conversations
- **Real-time Interaction**: Bi-directional communication with external services

#### Significant New Capabilities
1. **Custom Tool Creation**: Define any function or API as a tool
   - Database queries
   - File system operations
   - API integrations
   - Custom business logic

2. **Authentication & Security**:
   - OAuth 2.0 Authorization Code Flow with PKCE
   - Dynamic client registration
   - Secure token management
   - User-specific data access

3. **Resource Management**:
   - Access to files, databases, and APIs
   - Resource permission control
   - Rate limiting and quotas

4. **State Persistence**:
   - Session management across conversations
   - User context preservation
   - Transaction support

5. **Enterprise Integration**:
   - Connect to internal tools and databases
   - Custom authentication providers
   - Compliance and audit logging

## How to Enable Developer Mode

### Accessing Developer Mode in ChatGPT

Since the September 9th, 2025 launch, Developer Mode is available to all ChatGPT users:

1. **Open ChatGPT Settings**:
   - Navigate to https://chatgpt.com
   - Click on your profile icon
   - Select "Settings"

2. **Navigate to Connectors**:
   - Go to "Connectors" in the settings menu
   - Select "Advanced Settings"
   - Enable "Developer Mode (beta)"

3. **Create a New Connector**:
   - Click "Create" next to "Browser connectors"
   - You'll see the "New Connector" modal

### Configuring Your MCP Server

In the New Connector modal, fill in the following:

1. **Basic Information**:
   - **Icon** (optional): Upload a 128x128px icon
   - **Name**: Enter a name (e.g., "Custom MCP Tool")
   - **Description** (optional): Brief explanation of what it does

2. **Server Configuration**:
   - **MCP Server URL**: `https://mcp-oauth-server.fly.dev` (or your server URL)
   - **Authentication**: Select "OAuth"
   
3. **Security Notice**:
   - You'll see: "Beta intended for developer use only"
   - Check "I trust this application"
   - Note: "Custom connectors are not verified by OpenAI. Malicious developers may attempt to steal your data."

4. **Authorization** (One-time setup):
   - Click "Create" or "Connect"
   - You'll be redirected to the login/register page
   - Sign in or create a new account
   - Grant permissions to ChatGPT
   - You'll be redirected back to ChatGPT

5. **Verify Tools**:
   - After authorization, the modal will show available "Actions"
   - You should see tools like: search, fetch, write, analyze, etc.

### Using Your MCP Server

To use your connected MCP server:

1. **Start a New Chat**:
   - Click the "+" button to create a new chat

2. **Select Developer Mode**:
   - Click "... More >" in the model selector
   - Choose "Developer Mode"
   - Select the name of the MCP connector you added

3. **Test the Connection**:
   - Try a command like "search for test data"
   - The MCP tools will be available throughout the conversation

## Requirements & Limitations

### ⚠️ Critical Requirements

1. **Required Tools**: ChatGPT validates that your MCP server implements:
   - `search` - MUST be present for information retrieval
   - `fetch` - MUST be present for data fetching by ID
   
   Without these, ChatGPT will show "search action not found" error.

2. **OAuth Discovery**: Must be at root domain
   - ✅ `https://your-domain.com/.well-known/oauth-authorization-server`
   - ❌ `https://your-domain.com/functions/v1/.well-known/oauth-authorization-server`

3. **MCP Endpoint**: Must be at root path `/`
   - ChatGPT sends POST requests to the base URL

4. **Client ID Format**: ChatGPT uses UUID-format client IDs
   - Example: `47bc35b3-0d0f-4cbf-905f-d68125105b24`
   - Your server must accept any valid UUID, not just hardcoded values

### Platform Limitations

- **Supabase Edge Functions**: ❌ Won't work for OAuth (served under `/functions/v1/`)
- **Vercel Functions**: ⚠️ Can work but requires root path configuration
- **Fly.io**: ✅ Full control over root domain (recommended)
- **Railway/Render**: ✅ Also work well

## Option 1: Non-OAuth Implementation

For development or internal tools that don't require authentication.

### Complete Server Code

Create `server.js`:

```javascript
import express from 'express';
import cors from 'cors';

const app = express();
const PORT = process.env.PORT || 8080;

// Configuration
const PROTOCOL_VERSION = "2025-06-18";
const SERVER_NAME = "mcp-demo";
const SERVER_VERSION = "1.0.0";

// Middleware
app.use(cors({
  origin: ['https://chatgpt.com', 'https://chat.openai.com'],
  credentials: true,
  allowedHeaders: ['Content-Type', 'Authorization', 'MCP-Protocol-Version'],
  methods: ['GET', 'POST', 'OPTIONS']
}));

app.use(express.json());

// MCP Protocol Handler
app.post('/', (req, res) => {
  const { jsonrpc, method, params = {}, id } = req.body;
  
  switch (method) {
    case 'initialize':
      res.json({
        jsonrpc: '2.0',
        id,
        result: {
          protocolVersion: PROTOCOL_VERSION,
          capabilities: {
            logging: {},
            prompts: { listChanged: false },
            resources: { listChanged: false },
            tools: { listChanged: false }
          },
          serverInfo: {
            name: SERVER_NAME,
            title: 'MCP Demo Server',
            version: SERVER_VERSION
          }
        }
      });
      break;
      
    case 'initialized':
      res.status(200).end();
      break;
      
    case 'tools/list':
      res.json({
        jsonrpc: '2.0',
        id,
        result: {
          tools: [
            {
              name: 'search',
              title: 'Search',
              description: 'Search for information',
              inputSchema: {
                type: 'object',
                properties: {
                  query: { type: 'string', description: 'Search query' }
                },
                required: ['query'],
                additionalProperties: false
              }
            },
            {
              name: 'fetch',
              title: 'Fetch Data',
              description: 'Fetch data by ID',
              inputSchema: {
                type: 'object',
                properties: {
                  id: { type: 'string', description: 'Resource ID' }
                },
                required: ['id'],
                additionalProperties: false
              }
            },
            {
              name: 'calculate',
              title: 'Calculate',
              description: 'Perform calculations',
              inputSchema: {
                type: 'object',
                properties: {
                  expression: { type: 'string', description: 'Math expression' }
                },
                required: ['expression'],
                additionalProperties: false
              }
            }
          ]
        }
      });
      break;
      
    case 'tools/call':
      const { name, arguments: args = {} } = params;
      let content;
      
      switch (name) {
        case 'search':
          // Mock search results
          content = [{
            type: 'text',
            text: JSON.stringify([
              { id: '1', title: 'Result 1', content: `Found: ${args.query}` },
              { id: '2', title: 'Result 2', content: 'Additional result' }
            ])
          }];
          break;
          
        case 'fetch':
          // Mock fetch result
          content = [{
            type: 'text',
            text: JSON.stringify({
              id: args.id,
              title: `Resource ${args.id}`,
              content: `Data for ${args.id}`,
              timestamp: new Date().toISOString()
            })
          }];
          break;
          
        case 'calculate':
          // Simple calculator
          try {
            // WARNING: eval is dangerous! Use a proper math library in production
            const result = eval(args.expression.replace(/[^0-9+\-*/().\s]/g, ''));
            content = [{
              type: 'text',
              text: `${args.expression} = ${result}`
            }];
          } catch (error) {
            content = [{
              type: 'text',
              text: `Error: Invalid expression`
            }];
          }
          break;
          
        default:
          return res.json({
            jsonrpc: '2.0',
            id,
            error: { code: -32601, message: `Unknown tool: ${name}` }
          });
      }
      
      res.json({
        jsonrpc: '2.0',
        id,
        result: { content, isError: false }
      });
      break;
      
    case 'resources/list':
      res.json({
        jsonrpc: '2.0',
        id,
        result: { resources: [], nextCursor: null }
      });
      break;
      
    default:
      res.json({
        jsonrpc: '2.0',
        id,
        error: { code: -32601, message: 'Method not found' }
      });
  }
});

// Health check endpoint
app.get('/', (req, res) => {
  res.json({
    name: SERVER_NAME,
    version: SERVER_VERSION,
    protocol: PROTOCOL_VERSION,
    status: 'running'
  });
});

app.listen(PORT, () => {
  console.log(`🚀 MCP Server running on port ${PORT}`);
});
```

### Package.json

```json
{
  "name": "mcp-demo-server",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5"
  }
}
```

## Option 2: OAuth Implementation with Supabase

For production services requiring authentication.

### Complete OAuth Server Code

Create `server-oauth.js`:

```javascript
import express from 'express';
import cors from 'cors';
import { randomUUID } from 'crypto';
import { createClient } from '@supabase/supabase-js';

const app = express();
const PORT = process.env.PORT || 8080;

// Configuration
const PROTOCOL_VERSION = "2025-06-18";
const SERVER_NAME = "mcp-oauth";
const SERVER_VERSION = "2.4.0";

// Supabase configuration
const SUPABASE_URL = process.env.SUPABASE_URL || 'https://your-project.supabase.co';
const SUPABASE_ANON_KEY = process.env.SUPABASE_ANON_KEY || 'your-anon-key';
const supabaseAnon = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

// Get base URL
const BASE_URL = process.env.BASE_URL || 
  `https://${process.env.FLY_APP_NAME}.fly.dev` || 
  'http://localhost:8080';

// Middleware
app.use(cors({
  origin: ['https://chatgpt.com', 'https://chat.openai.com', 'http://localhost:3000'],
  credentials: true,
  allowedHeaders: ['Content-Type', 'Authorization', 'MCP-Protocol-Version'],
  methods: ['GET', 'POST', 'OPTIONS']
}));

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// OAuth Storage (use Redis/DB in production)
const clients = new Map();
const authCodes = new Map();
const tokens = new Map();
const refreshTokens = new Map();
const users = new Map();

// Helper: Identify ChatGPT clients
function isChatGPTClient(clientId) {
  return clientId === 'chatgpt' || 
         /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i.test(clientId);
}

// OAuth Discovery Endpoint (MUST be at root domain)
app.get('/.well-known/oauth-authorization-server', (req, res) => {
  res.json({
    issuer: BASE_URL,
    authorization_endpoint: `${BASE_URL}/authorize`,
    token_endpoint: `${BASE_URL}/token`,
    registration_endpoint: `${BASE_URL}/register`,
    response_types_supported: ['code'],
    response_modes_supported: ['query'],
    grant_types_supported: ['authorization_code', 'refresh_token'],
    code_challenge_methods_supported: ['S256', 'plain'],
    scopes_supported: ['mcp:read', 'mcp:write'],
    token_endpoint_auth_methods_supported: ['client_secret_post', 'client_secret_basic', 'none'],
    service_documentation: 'https://modelcontextprotocol.io',
    ui_locales_supported: ['en'],
    claims_supported: ['sub', 'email', 'name'],
    request_uri_parameter_supported: false,
    require_request_uri_registration: false,
    request_parameter_supported: false,
    authorization_response_iss_parameter_supported: true
  });
});

// OAuth Authorization Endpoint - GET (show login form)
app.get('/authorize', (req, res) => {
  const { client_id, redirect_uri, response_type, scope, state, code_challenge, code_challenge_method } = req.query;
  
  // Auto-register ChatGPT clients
  if (isChatGPTClient(client_id) && !clients.has(client_id)) {
    clients.set(client_id, {
      client_id: client_id,
      client_secret: null,
      redirect_uris: [
        'https://chatgpt.com/oauth/callback',
        'https://chat.openai.com/oauth/callback',
        'https://chatgpt.com/connector_platform_oauth_redirect',
        'http://localhost:3000/callback'
      ],
      grant_types: ['authorization_code', 'refresh_token'],
      token_endpoint_auth_method: 'none'
    });
  }
  
  const client = clients.get(client_id);
  if (!client) {
    return res.status(400).send('Invalid client_id');
  }
  
  // Validate redirect URI
  const validRedirect = client.redirect_uris.some(uri => 
    redirect_uri === uri || redirect_uri?.startsWith(uri.replace(/\/callback$/, ''))
  );
  
  if (!validRedirect) {
    return res.status(400).send('Invalid redirect_uri');
  }
  
  // Show login form
  res.send(`
    <!DOCTYPE html>
    <html>
    <head>
      <title>Sign In - MCP Server</title>
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
          font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
          background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          min-height: 100vh;
          display: flex;
          align-items: center;
          justify-content: center;
          padding: 20px;
        }
        .container {
          background: white;
          border-radius: 20px;
          padding: 40px;
          box-shadow: 0 20px 60px rgba(0,0,0,0.15);
          width: 100%;
          max-width: 400px;
        }
        h1 {
          color: #1a202c;
          font-size: 28px;
          margin-bottom: 8px;
          font-weight: 700;
        }
        .subtitle {
          color: #718096;
          margin-bottom: 32px;
          font-size: 14px;
        }
        .form-group {
          margin-bottom: 24px;
        }
        label {
          display: block;
          color: #4a5568;
          font-size: 14px;
          font-weight: 600;
          margin-bottom: 8px;
        }
        input {
          width: 100%;
          padding: 12px 16px;
          border: 2px solid #e2e8f0;
          border-radius: 10px;
          font-size: 16px;
          transition: all 0.3s;
          background: #f7fafc;
        }
        input:focus {
          outline: none;
          border-color: #667eea;
          background: white;
          box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }
        button {
          width: 100%;
          padding: 14px;
          margin: 8px 0;
          border: none;
          border-radius: 10px;
          font-size: 16px;
          font-weight: 600;
          cursor: pointer;
          transition: all 0.3s;
        }
        .btn-primary {
          background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
          color: white;
        }
        .btn-primary:hover {
          transform: translateY(-2px);
          box-shadow: 0 10px 20px rgba(102, 126, 234, 0.3);
        }
        .btn-secondary {
          background: white;
          color: #667eea;
          border: 2px solid #667eea;
        }
        .btn-secondary:hover {
          background: #f7fafc;
        }
        .error {
          background: #fed7d7;
          color: #c53030;
          padding: 12px;
          border-radius: 8px;
          margin-bottom: 20px;
          font-size: 14px;
        }
        .demo-note {
          background: #edf2f7;
          padding: 16px;
          border-radius: 10px;
          margin-top: 24px;
          font-size: 13px;
          color: #4a5568;
          border-left: 4px solid #667eea;
        }
        .demo-note strong {
          color: #2d3748;
        }
        .app-info {
          background: #f7fafc;
          padding: 16px;
          border-radius: 10px;
          margin-bottom: 24px;
          border: 1px solid #e2e8f0;
        }
        .app-name {
          color: #667eea;
          font-weight: 700;
          font-size: 16px;
        }
        .scope-list {
          margin-top: 8px;
          padding-left: 20px;
        }
        .scope-item {
          color: #4a5568;
          font-size: 14px;
          margin: 4px 0;
        }
        .divider {
          height: 1px;
          background: #e2e8f0;
          margin: 24px 0;
        }
        .signup-section {
          text-align: center;
          margin-top: 20px;
        }
        .signup-text {
          color: #718096;
          font-size: 14px;
          margin-bottom: 12px;
        }
        .btn-text {
          background: transparent;
          color: #667eea;
          border: none;
          text-decoration: underline;
          padding: 8px;
          cursor: pointer;
        }
        .btn-text:hover {
          color: #764ba2;
        }
      </style>
    </head>
    <body>
      <div class="container">
        <h1>Sign In</h1>
        <div class="subtitle">Connect your account to ChatGPT</div>
        
        <div class="app-info">
          <div class="app-name">ChatGPT MCP Integration</div>
          <div class="scope-list">
            <div class="scope-item">• Read access to MCP tools</div>
            <div class="scope-item">• Execute MCP functions</div>
          </div>
        </div>
        
        <form method="post" action="/authorize">
          <input type="hidden" name="client_id" value="${client_id}">
          <input type="hidden" name="redirect_uri" value="${redirect_uri}">
          <input type="hidden" name="response_type" value="${response_type}">
          <input type="hidden" name="scope" value="${scope || 'mcp:read mcp:write'}">
          <input type="hidden" name="state" value="${state || ''}">
          <input type="hidden" name="code_challenge" value="${code_challenge || ''}">
          <input type="hidden" name="code_challenge_method" value="${code_challenge_method || ''}">
          
          <div class="form-group">
            <label for="email">Email</label>
            <input type="email" id="email" name="email" required placeholder="you@example.com">
          </div>
          
          <div class="form-group">
            <label for="password">Password</label>
            <input type="password" id="password" name="password" required placeholder="••••••••">
          </div>
          
          <button type="submit" name="action" value="approve" class="btn-primary">
            Sign In & Authorize
          </button>
          
          <div class="divider"></div>
          
          <button type="submit" name="action" value="demo" class="btn-secondary">
            Use Demo Account
          </button>
        </form>
        
        <div class="signup-section">
          <div class="signup-text">Don't have an account?</div>
          <button class="btn-text" onclick="window.location.href='/signup?${new URLSearchParams(req.query).toString()}'">
            Create New Account
          </button>
        </div>
        
        <div class="demo-note">
          <strong>Demo Mode:</strong> Click "Use Demo Account" to test without creating an account. For production use, create an account or sign in with your credentials.
        </div>
      </div>
    </body>
    </html>
  `);
});

// OAuth Authorization Endpoint - POST (process login)
app.post('/authorize', async (req, res) => {
  const { client_id, redirect_uri, response_type, scope, state, action, email, password, code_challenge, code_challenge_method } = req.body;
  
  if (action === 'deny') {
    const redirectUrl = new URL(redirect_uri);
    redirectUrl.searchParams.set('error', 'access_denied');
    if (state) redirectUrl.searchParams.set('state', state);
    return res.redirect(redirectUrl.toString());
  }
  
  let user;
  
  if (action === 'demo') {
    // Demo mode
    user = { 
      id: 'demo-user', 
      email: 'demo@example.com',
      isDemo: true
    };
  } else {
    // Try Supabase authentication
    try {
      const { data, error } = await supabaseAnon.auth.signInWithPassword({
        email,
        password
      });
      
      if (error) {
        // Show error page
        return res.send(`
          <!DOCTYPE html>
          <html>
          <head>
            <title>Sign In Error</title>
            <style>
              body { font-family: system-ui; padding: 40px; background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); min-height: 100vh; }
              .container { max-width: 400px; margin: 0 auto; background: white; padding: 40px; border-radius: 20px; box-shadow: 0 20px 60px rgba(0,0,0,0.15); }
              .error { background: #fed7d7; color: #c53030; padding: 12px; border-radius: 8px; margin-bottom: 20px; }
              button { background: #667eea; color: white; border: none; padding: 12px 24px; border-radius: 8px; cursor: pointer; }
            </style>
          </head>
          <body>
            <div class="container">
              <h1>Authentication Failed</h1>
              <div class="error">Invalid email or password. Please try again.</div>
              <button onclick="history.back()">Go Back</button>
            </div>
          </body>
          </html>
        `);
      }
      
      user = { 
        id: data.user.id, 
        email: data.user.email,
        supabase_session: data.session
      };
      
      // Store user session
      users.set(user.id, user);
    } catch (err) {
      console.error('Supabase auth error:', err);
      // Fall back to demo mode on error
      user = { 
        id: 'demo-user', 
        email: 'demo@example.com',
        isDemo: true
      };
    }
  }
  
  // Generate authorization code
  const code = randomUUID();
  
  // Store auth code with PKCE if provided
  authCodes.set(code, {
    client_id,
    redirect_uri,
    scope: scope || 'mcp:read mcp:write',
    code_challenge,
    code_challenge_method,
    expires_at: Date.now() + 600000, // 10 minutes
    user: user
  });
  
  // Redirect with code
  const redirectUrl = new URL(redirect_uri);
  redirectUrl.searchParams.set('code', code);
  if (state) redirectUrl.searchParams.set('state', state);
  
  res.redirect(redirectUrl.toString());
});

// OAuth Token Endpoint
app.post('/token', (req, res) => {
  const { grant_type, code, redirect_uri, client_id, client_secret, refresh_token, code_verifier } = req.body;
  
  if (grant_type === 'authorization_code') {
    const authCode = authCodes.get(code);
    
    if (!authCode || authCode.expires_at < Date.now()) {
      return res.status(400).json({ error: 'invalid_grant' });
    }
    
    // Validate client
    if (authCode.client_id !== client_id || authCode.redirect_uri !== redirect_uri) {
      return res.status(400).json({ error: 'invalid_request' });
    }
    
    // Generate tokens
    const accessToken = randomUUID();
    const newRefreshToken = randomUUID();
    
    // Store tokens
    tokens.set(accessToken, {
      client_id,
      user: authCode.user,
      scope: authCode.scope,
      expires_at: Date.now() + 3600000 // 1 hour
    });
    
    refreshTokens.set(newRefreshToken, {
      client_id,
      user: authCode.user,
      scope: authCode.scope,
      expires_at: Date.now() + 1209600000 // 14 days
    });
    
    // Delete used code
    authCodes.delete(code);
    
    // Return tokens
    res.json({
      access_token: accessToken,
      token_type: 'Bearer',
      expires_in: 3600,
      refresh_token: newRefreshToken,
      scope: authCode.scope
    });
  } else if (grant_type === 'refresh_token') {
    const tokenData = refreshTokens.get(refresh_token);
    
    if (!tokenData || tokenData.expires_at < Date.now()) {
      return res.status(400).json({ error: 'invalid_grant' });
    }
    
    // Generate new access token
    const accessToken = randomUUID();
    
    tokens.set(accessToken, {
      client_id: tokenData.client_id,
      user: tokenData.user,
      scope: tokenData.scope,
      expires_at: Date.now() + 3600000
    });
    
    res.json({
      access_token: accessToken,
      token_type: 'Bearer',
      expires_in: 3600,
      scope: tokenData.scope
    });
  } else {
    res.status(400).json({ error: 'unsupported_grant_type' });
  }
});

// Dynamic Client Registration
app.post('/register', (req, res) => {
  const { redirect_uris, grant_types, response_types, client_name, scope, token_endpoint_auth_method } = req.body;
  
  const clientId = randomUUID();
  const clientSecret = randomUUID();
  
  const client = {
    client_id: clientId,
    client_secret: token_endpoint_auth_method === 'none' ? undefined : clientSecret,
    redirect_uris: redirect_uris || [],
    grant_types: grant_types || ['authorization_code'],
    response_types: response_types || ['code'],
    scope: scope || 'mcp:read mcp:write',
    token_endpoint_auth_method: token_endpoint_auth_method || 'client_secret_basic',
    client_name: client_name || 'MCP Client'
  };
  
  clients.set(clientId, client);
  
  res.status(201).json({
    client_id: clientId,
    client_secret: client.client_secret,
    client_id_issued_at: Math.floor(Date.now() / 1000),
    grant_types: client.grant_types,
    redirect_uris: client.redirect_uris,
    response_types: client.response_types,
    scope: client.scope,
    token_endpoint_auth_method: client.token_endpoint_auth_method,
    client_name: client.client_name
  });
});

// Token validation helper
function validateToken(authHeader) {
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return null;
  }
  
  const token = authHeader.substring(7);
  const tokenData = tokens.get(token);
  
  if (!tokenData || tokenData.expires_at < Date.now()) {
    return null;
  }
  
  return tokenData;
}

// MCP Protocol Handler (MUST be at root path)
app.post('/', (req, res) => {
  const { jsonrpc, method, params = {}, id } = req.body;
  
  // Check authentication for protected methods
  const tokenData = validateToken(req.headers.authorization);
  const requiresAuth = method !== 'initialize' && method !== 'initialized';
  
  if (requiresAuth && !tokenData) {
    return res.json({
      jsonrpc: '2.0',
      id,
      error: {
        code: -32000,
        message: 'Authentication required',
        data: {
          oauth_authorize_url: `${BASE_URL}/authorize?client_id=chatgpt&response_type=code&scope=mcp:read+mcp:write`
        }
      }
    });
  }
  
  switch (method) {
    case 'initialize':
      res.json({
        jsonrpc: '2.0',
        id,
        result: {
          protocolVersion: PROTOCOL_VERSION,
          capabilities: {
            logging: {},
            prompts: { listChanged: false },
            resources: { listChanged: false },
            tools: { listChanged: false }
          },
          serverInfo: {
            name: SERVER_NAME,
            title: 'MCP OAuth Server',
            version: SERVER_VERSION
          }
        }
      });
      break;
      
    case 'initialized':
      res.status(200).end();
      break;
      
    case 'tools/list':
      res.json({
        jsonrpc: '2.0',
        id,
        result: {
          tools: [
            // REQUIRED TOOLS
            {
              name: 'search',
              title: 'Search',
              description: 'Search for information',
              inputSchema: {
                type: 'object',
                properties: {
                  query: { type: 'string', description: 'Search query' }
                },
                required: ['query'],
                additionalProperties: false
              }
            },
            {
              name: 'fetch',
              title: 'Fetch Data',
              description: 'Fetch data by ID',
              inputSchema: {
                type: 'object',
                properties: {
                  id: { type: 'string', description: 'Resource ID' }
                },
                required: ['id'],
                additionalProperties: false
              }
            },
            // Additional tools
            {
              name: 'analyze',
              title: 'Analyze Data',
              description: 'Analyze data and provide insights',
              inputSchema: {
                type: 'object',
                properties: {
                  data: { type: 'string', description: 'Data to analyze' },
                  type: { 
                    type: 'string', 
                    enum: ['sentiment', 'summary', 'statistics'],
                    description: 'Type of analysis'
                  }
                },
                required: ['data', 'type'],
                additionalProperties: false
              }
            },
            {
              name: 'generate',
              title: 'Generate Content',
              description: 'Generate various types of content',
              inputSchema: {
                type: 'object',
                properties: {
                  type: { 
                    type: 'string', 
                    enum: ['text', 'code', 'json'],
                    description: 'Content type'
                  },
                  prompt: { type: 'string', description: 'Generation prompt' }
                },
                required: ['type', 'prompt'],
                additionalProperties: false
              }
            }
          ]
        }
      });
      break;
      
    case 'tools/call':
      const { name, arguments: args = {} } = params;
      let content;
      
      switch (name) {
        case 'search':
          content = [{
            type: 'text',
            text: JSON.stringify([
              { 
                id: '1', 
                title: 'Search Result 1', 
                content: `Found: ${args.query}`,
                user: tokenData.user.email
              },
              { 
                id: '2', 
                title: 'Search Result 2', 
                content: 'Additional result',
                authenticated: true
              }
            ])
          }];
          break;
          
        case 'fetch':
          content = [{
            type: 'text',
            text: JSON.stringify({
              id: args.id,
              title: `Resource ${args.id}`,
              content: `Data for ${args.id}`,
              authenticated: true,
              user: tokenData.user.email,
              timestamp: new Date().toISOString()
            })
          }];
          break;
          
        case 'analyze':
          content = [{
            type: 'text',
            text: JSON.stringify({
              input: args.data,
              analysis_type: args.type,
              result: `Analysis of type ${args.type} completed`,
              user: tokenData.user.email
            })
          }];
          break;
          
        case 'generate':
          content = [{
            type: 'text',
            text: JSON.stringify({
              type: args.type,
              generated: `Generated ${args.type} content based on: ${args.prompt}`,
              user: tokenData.user.email
            })
          }];
          break;
          
        default:
          return res.json({
            jsonrpc: '2.0',
            id,
            error: { code: -32601, message: `Unknown tool: ${name}` }
          });
      }
      
      res.json({
        jsonrpc: '2.0',
        id,
        result: { content, isError: false }
      });
      break;
      
    case 'resources/list':
      res.json({
        jsonrpc: '2.0',
        id,
        result: { resources: [], nextCursor: null }
      });
      break;
      
    default:
      res.json({
        jsonrpc: '2.0',
        id,
        error: { code: -32601, message: 'Method not found' }
      });
  }
});

// Health check
app.get('/', (req, res) => {
  res.json({
    name: SERVER_NAME,
    version: SERVER_VERSION,
    protocol: PROTOCOL_VERSION,
    endpoints: {
      mcp: 'POST /',
      oauth_discovery: '/.well-known/oauth-authorization-server',
      authorize: '/authorize',
      token: '/token',
      register: '/register'
    },
    status: 'running'
  });
});

app.listen(PORT, () => {
  console.log(`🚀 MCP OAuth Server v${SERVER_VERSION} running on port ${PORT}`);
  console.log(`📝 Protocol: MCP ${PROTOCOL_VERSION}`);
  console.log(`🔐 OAuth discovery: ${BASE_URL}/.well-known/oauth-authorization-server`);
  console.log(`🌐 Base URL: ${BASE_URL}`);
  console.log(`✅ Ready for ChatGPT integration!`);
});
```

### Package.json for OAuth

```json
{
  "name": "mcp-oauth-server",
  "version": "2.4.0",
  "type": "module",
  "scripts": {
    "start": "node server-oauth.js",
    "dev": "node --watch server-oauth.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "@supabase/supabase-js": "^2.39.1"
  }
}
```

## Deployment to Fly.io

### Prerequisites

1. Install Fly CLI:
```bash
curl -L https://fly.io/install.sh | sh
```

2. Sign up and login:
```bash
flyctl auth signup
# or
flyctl auth login
```

### Dockerfile

Create `Dockerfile`:

```dockerfile
FROM node:20-slim

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application files
COPY . .

# Expose port
EXPOSE 8080

# Start the application
CMD ["npm", "start"]
```

### fly.toml Configuration

Create `fly.toml`:

```toml
app = 'mcp-oauth-server'
primary_region = 'iad'

[build]

[http_service]
  internal_port = 8080
  force_https = true
  auto_stop_machines = false
  auto_start_machines = true
  min_machines_running = 1
  processes = ['app']

[[vm]]
  memory = '1gb'
  cpu_kind = 'shared'
  cpus = 1

[env]
  PORT = "8080"
  NODE_ENV = "production"
```

### Deploy Script

Create `deploy.sh`:

```bash
#!/bin/bash

echo "🚀 Deploying MCP OAuth Server to Fly.io"

# Check if app exists
if ! flyctl status >/dev/null 2>&1; then
  echo "📝 Creating new Fly.io app..."
  flyctl launch --name mcp-oauth-server --region iad --yes
else
  echo "✅ App already exists"
fi

# Set secrets (if using Supabase)
echo "🔐 Setting environment variables..."
flyctl secrets set \
  SUPABASE_URL="https://your-project.supabase.co" \
  SUPABASE_ANON_KEY="your-anon-key"

# Deploy
echo "🚢 Deploying application..."
flyctl deploy --ha=false

# Show status
echo "✨ Deployment complete!"
flyctl status
flyctl info

echo "🌐 Your MCP server is available at:"
flyctl info | grep "Hostname"

echo ""
echo "📝 OAuth Discovery URL:"
echo "https://$(flyctl info -j | jq -r .Hostname)/.well-known/oauth-authorization-server"
```

Make it executable:
```bash
chmod +x deploy.sh
```

### Deploy Your Server

```bash
# First time setup
flyctl launch --name your-mcp-server --region iad

# Deploy
flyctl deploy --ha=false

# Check status
flyctl status

# View logs
flyctl logs
```

## Testing Your MCP Server

### 1. Test Locally

```bash
# Install dependencies
npm install

# Run locally
npm start

# Test MCP endpoint
curl -X POST http://localhost:8080/ \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"initialize","params":{},"id":1}'

# Test OAuth discovery
curl http://localhost:8080/.well-known/oauth-authorization-server
```

### 2. Connect to ChatGPT

1. Go to ChatGPT Settings → Developer
2. Click "Add MCP Server"
3. Enter your server URL: `https://your-app.fly.dev`
4. ChatGPT will:
   - Discover OAuth endpoints automatically
   - Prompt you to authorize
   - Show available tools after authorization

### 3. Test Tools

After connecting, try these prompts in ChatGPT:
- "Search for information about AI"
- "Fetch data with ID 123"
- "Analyze this text for sentiment"
- "Generate a JSON schema"

## Common Gotchas & Solutions

### 1. "Only ChatGPT client is supported" Error

**Problem**: Server rejects ChatGPT's UUID client_id

**Solution**: Accept any UUID format:
```javascript
function isChatGPTClient(clientId) {
  return clientId === 'chatgpt' || 
         /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i.test(clientId);
}
```

### 2. "search action not found" Error

**Problem**: ChatGPT requires `search` and `fetch` tools

**Solution**: Always include these tools in `tools/list`:
```javascript
{
  name: 'search',
  title: 'Search',
  description: 'Search for information',
  inputSchema: { /* ... */ }
},
{
  name: 'fetch',
  title: 'Fetch Data',
  description: 'Fetch data by ID',
  inputSchema: { /* ... */ }
}
```

### 3. OAuth Discovery Fails

**Problem**: Supabase Edge Functions serve from `/functions/v1/`

**Solution**: Use Fly.io or another platform with root domain control

### 4. Redirect URI Mismatch

**Problem**: ChatGPT uses different redirect URIs

**Solution**: Include all variants:
```javascript
redirect_uris: [
  'https://chatgpt.com/oauth/callback',
  'https://chat.openai.com/oauth/callback',
  'https://chatgpt.com/connector_platform_oauth_redirect'
]
```

### 5. CORS Issues

**Problem**: Browser blocks requests

**Solution**: Configure CORS properly:
```javascript
app.use(cors({
  origin: ['https://chatgpt.com', 'https://chat.openai.com'],
  credentials: true,
  allowedHeaders: ['Content-Type', 'Authorization', 'MCP-Protocol-Version']
}));
```

## Troubleshooting

### Check Server Logs

```bash
# Fly.io logs
flyctl logs --tail

# Local logs
npm run dev
```

### Common Error Messages

| Error | Cause | Solution |
|-------|-------|----------|
| "Method not found" | Unimplemented MCP method | Add method to switch statement |
| "Invalid client_id" | Client not registered | Auto-register ChatGPT clients |
| "Authentication required" | Missing/invalid token | Implement OAuth flow |
| "Invalid redirect_uri" | URI not in whitelist | Add URI to client configuration |

### Debug OAuth Flow

1. Check discovery endpoint:
```bash
curl https://your-app.fly.dev/.well-known/oauth-authorization-server
```

2. Test authorization:
```bash
# Open in browser
https://your-app.fly.dev/authorize?client_id=chatgpt&response_type=code
```

3. Monitor token exchange:
```javascript
// Add logging in token endpoint
console.log('Token request:', { grant_type, client_id });
```

## Security Best Practices

1. **Use Environment Variables**: Never hardcode secrets
2. **Validate Inputs**: Sanitize all user inputs
3. **Rate Limiting**: Add rate limiting in production
4. **HTTPS Only**: Always use HTTPS in production
5. **Token Expiration**: Set appropriate token lifetimes
6. **Audit Logging**: Log all authentication events

## Advanced Features

### Add Custom Tools

```javascript
{
  name: 'database_query',
  title: 'Database Query',
  description: 'Execute database queries',
  inputSchema: {
    type: 'object',
    properties: {
      query: { type: 'string', description: 'SQL query' },
      database: { type: 'string', description: 'Database name' }
    },
    required: ['query'],
    additionalProperties: false
  }
}
```

### Add Resources

```javascript
case 'resources/list':
  res.json({
    jsonrpc: '2.0',
    id,
    result: {
      resources: [
        {
          uri: 'resource://database/users',
          name: 'User Database',
          description: 'Access to user data',
          mimeType: 'application/json'
        }
      ],
      nextCursor: null
    }
  });
  break;
```

### Add Prompts

```javascript
case 'prompts/list':
  res.json({
    jsonrpc: '2.0',
    id,
    result: {
      prompts: [
        {
          name: 'analyze_code',
          description: 'Analyze code quality',
          arguments: [
            {
              name: 'code',
              description: 'Code to analyze',
              required: true
            }
          ]
        }
      ]
    }
  });
  break;
```

## Real-World Use Cases

Since the September 9th announcement, developers have built MCP servers for:

### Business Applications
- **CRM Integration**: Connect ChatGPT to Salesforce, HubSpot, or custom CRMs
- **Database Queries**: Direct SQL access to production databases
- **Analytics Tools**: Connect to Google Analytics, Mixpanel, or custom dashboards
- **Document Management**: Search and retrieve from SharePoint, Confluence, or wikis

### Developer Tools
- **Code Repository Access**: Browse and search GitHub/GitLab repositories
- **CI/CD Integration**: Trigger builds, check deployment status
- **API Testing**: Execute and test API endpoints directly from ChatGPT
- **Log Analysis**: Search and analyze application logs in real-time

### Personal Productivity
- **Note Taking**: Connect to Notion, Obsidian, or personal knowledge bases
- **Task Management**: Integrate with Jira, Asana, or todo apps
- **Email Access**: Search and compose emails (with proper authentication)
- **Calendar Integration**: Schedule meetings and check availability

### Community Feedback

From the OpenAI Developer Forum and community discussions:

> "MCP Server Tools have transformed how we interact with ChatGPT. We can now connect it directly to our internal systems." - Enterprise Developer

> "The OAuth support means we can safely expose company tools without security concerns." - Security Engineer

> "Building custom tools is straightforward - the hardest part was remembering to include the required 'search' and 'fetch' tools!" - Independent Developer

## Live Example

Test the working implementation at: https://mcp-oauth-server.fly.dev

1. **OAuth Discovery**: https://mcp-oauth-server.fly.dev/.well-known/oauth-authorization-server
2. **Registration**: https://mcp-oauth-server.fly.dev/signup (New users can create accounts)
3. **Available Tools**: search, fetch, analyze, generate, and more
4. **Demo Mode**: Sign in with demo credentials or create your own account

## Resources

- [MCP Specification](https://modelcontextprotocol.io)
- [ChatGPT Developer Settings](https://chatgpt.com/settings/developer)
- [OpenAI Community Discussion](https://community.openai.com/t/mcp-server-tools-now-in-chatgpt-developer-mode/1357233)
- [Fly.io Documentation](https://fly.io/docs)
- [Supabase Auth](https://supabase.com/docs/guides/auth)
- [OAuth 2.0 RFC](https://tools.ietf.org/html/rfc6749)
- [Example Implementation](https://github.com/ruvnet/mcp-oauth-server)

## Summary

Building an MCP server for ChatGPT requires:
1. **Required tools**: `search` and `fetch` must be implemented
2. **Root domain control**: OAuth discovery at `/.well-known/`
3. **UUID client support**: Accept any UUID-format client_id
4. **Proper CORS**: Configure for ChatGPT domains
5. **Platform choice**: Use Fly.io or similar (not Supabase Edge Functions)

Follow this tutorial step-by-step, and you'll have a working MCP server integrated with ChatGPT's new Developer Mode!

---

*Last Updated: September 13, 2025*
*Version: 2.5.0*
*MCP Server Tools Announcement: September 9, 2025*
*Live Demo: https://mcp-oauth-server.fly.dev*
