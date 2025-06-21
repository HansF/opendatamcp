# Introduction

The goal of this repository is to provide a lightweight Model Context Protocol (MCP) server that queries the Opendatasoft Explore API. As described in the architecture document, the server is designed to remain stateless and easy to maintain:

> The MCP server acts as a lightweight, stateless gateway that allows Model Context Protocol (MCP) clients to query the [Opendatasoft Explore API](https://data.stad.gent/api/explore/v2.1). The primary goal is to expose a minimal set of features while keeping the codebase easy to maintain. Key objectives are:
> - Provide a simple MCP API for querying datasets and records from the Explore API.
> - Remain stateless so that the server can scale horizontally.
> - Keep dependencies and internal complexity to a minimum.
> - Support the MCP tools and prompts outlined in the **Everything MCP Server** documentation.
【F:docs/architecture.md†L5-L10】

# Project Overview

The project consists of two main parts:

1. **Opendatasoft API specification** (`server` directory) – contains OpenAPI and Swagger documentation for the Explore API.
2. **Everything MCP Server** (`docs/everything` directory) – a fully featured reference implementation used for testing MCP clients.

The architecture is organized around several layers:

```
[MCP Client] <--> [API Layer] --> [Query Processing] --> [Endpoint Communication] --> [Explore API]
                                                    ↘
                                                     [Data Management]
```
【F:docs/architecture.md†L22-L26】

# Prerequisites

- Node.js (version 18 or newer)
- npm
- Git

# Step-by-Step Guide

## 1. Clone the repository

```bash
git clone <repository-url>
cd opendatamcp
```

## 2. Inspect the documentation

Read `docs/architecture.md` for an overview of the system design. Pay special attention to the authentication requirements:

> **Responsibilities:** Support API key authentication as defined in the Swagger `apikey` security scheme. Keys are loaded from environment variables. No user-level authorization logic is handled beyond passing the key to the Explore API.
【F:docs/architecture.md†L47-L48】

## 3. Install dependencies

Navigate to the `docs/everything` folder and install dependencies:

```bash
cd docs/everything
npm install
```

*(If you encounter network errors during installation, ensure you have internet access or use an offline npm mirror.)*

## 4. Build the project

Compile the TypeScript source into the `dist` directory:

```bash
npm run build
```

## 5. Run the server

The entry point accepts a transport type. The default is `stdio` but you can also run SSE or Streamable HTTP.

```bash
# Default (stdio)
npm run start

# SSE
npm run start:sse

# Streamable HTTP
npm run start:streamableHttp
```

Alternatively, once installed globally you can use `npx`:

```bash
npx @modelcontextprotocol/server-everything            # stdio
npx @modelcontextprotocol/server-everything sse        # SSE
npx @modelcontextprotocol/server-everything streamableHttp
```
【F:docs/everything/README.md†L192-L215】

## 6. Provide an API key

Export your Opendatasoft API key before launching the server:

```bash
export ODS_API_KEY=<your-key>
```

The server reads this key from the environment and forwards it to the Explore API when processing requests.

## 7. Explore the API

Refer to `server/Docs.md` and `server/openapi.json` to understand available endpoints. Examples include listing datasets or fetching records from a dataset.

# Code Examples

Below is a simplified excerpt of the default server startup (`stdio.ts`):

```ts
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { createServer } from "./everything.js";

console.error('Starting default (STDIO) server...');

async function main() {
  const transport = new StdioServerTransport();
  const {server, cleanup} = createServer();

  await server.connect(transport);

  process.on("SIGINT", async () => {
    await cleanup();
    await server.close();
    process.exit(0);
  });
}

main().catch((error) => {
  console.error("Server error:", error);
  process.exit(1);
});
```
【F:docs/everything/stdio.ts†L1-L25】

The main entry (`index.ts`) chooses which transport to load based on the first command‑line argument:

```ts
const args = process.argv.slice(2);
const scriptName = args[0] || 'stdio';

async function run() {
    try {
        switch (scriptName) {
            case 'stdio':
                await import('./stdio.js');
                break;
            case 'sse':
                await import('./sse.js');
                break;
            case 'streamableHttp':
                await import('./streamableHttp.js');
                break;
            default:
                console.error(`Unknown script: ${scriptName}`);
                console.log('Available scripts:');
                console.log('- stdio');
                console.log('- sse');
                console.log('- streamableHttp');
                process.exit(1);
        }
    } catch (error) {
        console.error('Error running script:', error);
        process.exit(1);
    }
}

run();
```
【F:docs/everything/index.ts†L3-L37】

# Additional Resources

- [Architecture Overview](architecture.md)
- [Everything MCP Server README](everything/README.md)
- [Opendatasoft Explore API Docs](server/Docs.md)
- [OpenAPI Specification](server/openapi.json)

