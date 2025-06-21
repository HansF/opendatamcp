# MCP Server Architecture

## 1. Introduction

The MCP server acts as a lightweight, stateless gateway that allows Model Context Protocol (MCP) clients to query the [Opendatasoft Explore API](https://data.stad.gent/api/explore/v2.1). The primary goal is to expose a minimal set of features while keeping the codebase easy to maintain. Key objectives are:

- Provide a simple MCP API for querying datasets and records from the Explore API.
- Remain stateless so that the server can scale horizontally.
- Keep dependencies and internal complexity to a minimum.
- Support the MCP tools and prompts outlined in the **Everything MCP Server** documentation.

## 2. System Overview

The server is organized around a modular Node.js application with the following layers:

1. **API Layer** – Handles incoming MCP requests.
2. **Query Processing** – Parses requests and prepares calls to the Explore API.
3. **Endpoint Communication** – Executes HTTP requests to the Explore API.
4. **Data Management** – Caches recent query results and manages resources.
5. **Authentication & Authorization** – Attaches API keys as required.

```
[MCP Client] <--> [API Layer] --> [Query Processing] --> [Endpoint Communication] --> [Explore API]
                                                    ↘
                                                     [Data Management]
```
(Insert a system diagram here illustrating the components and their interactions.)

## 3. Component Design

### a. API Layer
- **Responsibilities:** Expose a `/mcp` endpoint (Streamable HTTP or stdio) that accepts MCP-compliant requests. Routes are minimal to keep the service lightweight.
- **Interactions:** For each request, the API layer invokes the Query Processing component and streams results back to the client.

### b. Query Processing
- **Responsibilities:** Validate MCP tool calls and prompts. Map incoming requests to appropriate Explore API operations. For example, a `getRecords` tool should translate to `GET /catalog/datasets/{dataset_id}/records`.
- **Interactions:** Uses the Endpoint Communication component to execute queries and processes results for MCP responses.

### c. Endpoint Communication
- **Responsibilities:** Perform HTTP calls to the Explore API using `fetch`/`axios` with a minimal wrapper. Ensures each request includes the API key parameter defined in the Swagger document.
- **Interactions:** Returns raw API responses to Query Processing. Handles rate limiting and retries.

### d. Data Management
- **Responsibilities:** Maintain a small in-memory cache of recent dataset and record queries to reduce latency. Handle temporary resources as described in the Everything server (e.g., `getResourceReference`). Because the server is stateless, the cache is best-effort and can be safely discarded between restarts.
- **Interactions:** Queried by Query Processing to serve repeated requests quickly.

### e. Authentication and Authorization
- **Responsibilities:** Support API key authentication as defined in the Swagger `apikey` security scheme. Keys are loaded from environment variables. No user-level authorization logic is handled beyond passing the key to the Explore API.

## 4. Data Flow

1. **Request** – The MCP client sends a tool invocation or prompt request to `/mcp`.
2. **Validation** – API Layer validates structure and forwards to Query Processing.
3. **Mapping** – Query Processing maps the tool to an Explore API path, assembles parameters, and checks the cache.
4. **External Call** – Endpoint Communication sends the HTTP request to the Explore API with the API key.
5. **Response Handling** – Results are formatted according to MCP (e.g., text, annotations, resources) and cached.
6. **Reply** – API Layer streams the final response back to the client.

(Insert a data flow diagram describing these steps.)

## 5. Scalability and Performance

- **Stateless Design:** Because no session information is stored, multiple instances can run behind a load balancer.
- **Caching:** A simple in-memory LRU cache avoids repeated external calls for identical queries. Cache entries expire quickly to remain lightweight.
- **Connection Pooling:** Reuse HTTP keep-alive connections to the Explore API for efficiency.
- **Batching:** When multiple clients request the same dataset simultaneously, the cache avoids redundant calls.

## 6. Security Considerations

- **API Key Protection:** Store the Explore API key in environment variables and never log it. Use HTTPS for all outbound requests.
- **Input Validation:** Sanitize all user input before constructing Explore API queries to avoid injection attacks.
- **Rate Limiting:** Respect the Explore API’s quota responses and optionally implement a local request limiter.
- **Error Masking:** Do not expose internal stack traces; return concise error messages to the client.

## 7. Error Handling and Logging

- **Structured Logs:** Output JSON logs that include request IDs, timestamps, and error details. Follow the logging approach shown in the Everything server, emitting periodic log messages for observability.
- **Graceful Errors:** Use try/catch blocks around external calls. Return standard MCP error messages with clear codes and descriptions.

## 8. Deployment and DevOps

- **Containerization:** Provide a small Dockerfile similar to the Everything server for easy deployment. Use multi-stage builds to keep the final image small.
- **CI/CD:** Automate builds and linting with a minimal pipeline. Since the server is stateless, deployments can be rolling updates with zero downtime.
- **Configuration:** All environment-specific values (API key, port, log level) are passed via environment variables.

## 9. Future Enhancements

- **Additional Tools:** Implement higher-level tools for common Explore API queries (e.g., dataset search).
- **Persistent Caching:** Add optional Redis or Memcached support for distributed caching across instances.
- **Metrics:** Export Prometheus metrics for request counts, latency, and cache hit rate.
- **Authentication Expansion:** Integrate OAuth if the Explore API or future endpoints require it.

