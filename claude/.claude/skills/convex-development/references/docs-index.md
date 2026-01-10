# Convex Documentation Index

When local references don't cover your use case, scan this index and WebFetch the relevant page from `https://docs.convex.dev`.

**Usage:** Find the topic below, then fetch `https://docs.convex.dev{path}` (e.g., `https://docs.convex.dev/auth/convex-auth`).

> For general information about Convex, read https://www.convex.dev/llms.txt

---

## Understanding

- [Convex Overview](/understanding.md): Introduction to Convex - the reactive database with TypeScript queries
- [Best Practices](/understanding/best-practices.md): Essential best practices for building scalable Convex applications
- [TypeScript](/understanding/best-practices/typescript.md): Move faster with end-to-end type safety
- [Dev workflow](/understanding/workflow.md): Development workflow from project creation to production deployment
- [The Zen of Convex](/understanding/zen.md): Convex best practices and design philosophy

## Quickstarts

- [Next.js Quickstart](/quickstart/nextjs.md): Add Convex to a Next.js project
- [React Quickstart](/quickstart/react.md): Add Convex to a React project
- [React Native Quickstart](/quickstart/react-native.md): Add Convex to a React Native Expo project
- [Node.js Quickstart](/quickstart/nodejs.md): Add Convex to a Node.js project
- [Python Quickstart](/quickstart/python.md): Add Convex to a Python project
- [Svelte Quickstart](/quickstart/svelte.md): Add Convex to a Svelte project
- [Vue Quickstart](/quickstart/vue.md): Add Convex to a Vue project
- [Bun Quickstart](/quickstart/bun.md): Add Convex to a Bun project
- [Remix Quickstart](/quickstart/remix.md): Add Convex to a Remix project
- [Nuxt Quickstart](/quickstart/nuxt.md): Add Convex to a Nuxt project
- [TanStack Start Quickstart](/quickstart/tanstack-start.md): Add Convex to a TanStack Start project
- [Android Kotlin Quickstart](/quickstart/android.md): Add Convex to an Android Kotlin project
- [iOS Swift Quickstart](/quickstart/swift.md): Add Convex to an iOS Swift project
- [Rust Quickstart](/quickstart/rust.md): Add Convex to a Rust project
- [Script Tag Quickstart](/quickstart/script-tag.md): Add Convex to any website

## Functions

- [Functions](/functions.md): Write functions to define your server behavior
- [Queries](/functions/query-functions.md): Fetch data from the database with caching and reactivity
- [Mutations](/functions/mutation-functions.md): Insert, update, and remove data from the database
- [Actions](/functions/actions.md): Call third-party services and external APIs from Convex
- [HTTP Actions](/functions/http-actions.md): Build HTTP APIs directly in Convex
- [Internal Functions](/functions/internal-functions.md): Functions that can only be called by other Convex functions
- [Argument and Return Value Validation](/functions/validation.md): Validate function arguments and return values
- [Error Handling](/functions/error-handling.md): Handle errors in Convex queries, mutations, and actions
- [Application Errors](/functions/error-handling/application-errors.md): Handle expected failures in Convex functions
- [Runtimes](/functions/runtimes.md): Differences between the Convex and Node.js runtimes
- [Bundling](/functions/bundling.md): How Convex bundles and optimizes your function code
- [Debugging](/functions/debugging.md): Debug Convex functions during development and production

## Database

- [Database](/database.md): Store JSON-like documents with a relational data model
- [Reading Data](/database/reading-data.md): Query and read data from Convex database tables
- [Writing Data](/database/writing-data.md): Insert, update, and delete data in Convex database tables
- [Document IDs](/database/document-ids.md): Create complex, relational data models using IDs
- [Schemas](/database/schemas.md): Schema validation with end-to-end TypeScript type safety
- [Data Types](/database/types.md): Supported data types in Convex documents
- [Indexes](/database/reading-data/indexes.md): Speed up queries with database indexes
- [Introduction to Indexes and Query Performance](/database/reading-data/indexes/indexes-and-query-perf.md): Effects of indexes on query performance
- [Filtering](/database/reading-data/filters.md): Filter documents in Convex queries
- [Paginated Queries](/database/pagination.md): Load paginated queries
- [Data Import & Export](/database/import-export.md): Import data from existing sources and export data
- [Data Import](/database/import-export/import.md): Import data into Convex
- [Data Export](/database/import-export/export.md): Export your data out of Convex
- [Backups](/database/backup-restore.md): Backup and restore your Convex data and files
- [OCC and Atomicity](/database/advanced/occ.md): Optimistic concurrency control and transaction atomicity
- [Schema Philosophy](/database/advanced/schema-philosophy.md): Convex schema design philosophy
- [System Tables](/database/advanced/system-tables.md): Access metadata for Convex built-in features

## Realtime

- [Realtime](/realtime.md): Building realtime apps with Convex

## Authentication

- [Authentication](/auth.md): Add authentication to your Convex app
- [Convex Auth](/auth/convex-auth.md): Built-in authentication for Convex applications
- [Convex & Clerk](/auth/clerk.md): Integrate Clerk authentication with Convex
- [Convex & Auth0](/auth/auth0.md): Integrate Auth0 authentication with Convex
- [Convex & WorkOS AuthKit](/auth/authkit.md): Integrate WorkOS AuthKit authentication with Convex
- [Storing Users in the Convex Database](/auth/database-auth.md): Store user information in your Convex database
- [Auth in Functions](/auth/functions-auth.md): Access user authentication in Convex functions
- [Debugging Authentication](/auth/debug.md): Troubleshoot authentication issues in Convex
- [Custom OIDC Provider](/auth/advanced/custom-auth.md): Integrate with any OpenID Connect identity provider
- [Custom JWT Provider](/auth/advanced/custom-jwt.md): Configure Convex to work with custom JWT providers

## Scheduling

- [Scheduling](/scheduling.md): Schedule functions to run once or repeatedly
- [Scheduled Functions](/scheduling/scheduled-functions.md): Schedule functions to run in the future
- [Cron Jobs](/scheduling/cron-jobs.md): Schedule recurring functions in Convex

## File Storage

- [File Storage](/file-storage.md): Store and serve files of any type
- [Uploading and Storing Files](/file-storage/upload-files.md): Upload files to Convex storage
- [Serving Files](/file-storage/serve-files.md): Serve files stored in Convex to users
- [Storing Generated Files](/file-storage/store-files.md): Store files generated in Convex actions
- [Accessing File Metadata](/file-storage/file-metadata.md): Access file metadata stored in Convex
- [Deleting Files](/file-storage/delete-files.md): Delete files stored in Convex

## Search

- [AI & Search](/search.md): Run search queries over your Convex documents
- [Full Text Search](/search/text-search.md): Run search queries over your Convex documents
- [Vector Search](/search/vector-search.md): Run vector search queries on embeddings

## Components

- [Components](/components.md): Self contained building blocks of your app
- [Using Components](/components/using.md): Using existing components
- [Authoring Components](/components/authoring.md): Creating new components
- [Understanding Components](/components/understanding.md): Understanding components

## AI Code Generation

- [AI Code Generation](/ai.md): How to use AI code generation effectively with Convex
- [Convex MCP Server](/ai/convex-mcp-server.md): Convex MCP server
- [Using Cursor with Convex](/ai/using-cursor.md): Tips for using Cursor with Convex
- [Using GitHub Copilot with Convex](/ai/using-github-copilot.md): Tips for using GitHub Copilot with Convex
- [Using Windsurf with Convex](/ai/using-windsurf.md): Tips for using Windsurf with Convex

## AI Agents

- [AI Agents](/agents.md): Building AI Agents with Convex
- [Getting Started with Agent](/agents/getting-started.md): Setting up the agent component
- [Agent Definition and Usage](/agents/agent-usage.md): Configuring and using the Agent class
- [Threads](/agents/threads.md): Group messages together in a conversation history
- [Messages](/agents/messages.md): Sending and receiving messages with an agent
- [Streaming](/agents/streaming.md): Streaming messages with an agent
- [Tools](/agents/tools.md): Using tool calls with the Agent component
- [LLM Context](/agents/context.md): Customizing the context provided to the Agent's LLM
- [RAG](/agents/rag.md): Retrieval-Augmented Generation with the Agent component
- [Files and Images](/agents/files.md): Working with images and files in the Agent component
- [Workflows](/agents/workflows.md): Defining long-lived workflows for the Agent component
- [Rate Limiting](/agents/rate-limiting.md): Control the rate of requests to your AI agent
- [Usage Tracking](/agents/usage-tracking.md): Tracking token usage of the Agent component
- [Debugging](/agents/debugging.md): Debugging the Agent component
- [Playground](/agents/playground.md): Test, debug, and develop with the agent
- [Human Agents](/agents/human-agents.md): Saving messages from a human as an agent

## Testing

- [Testing](/testing.md): Testing your backend
- [convex-test](/testing/convex-test.md): Mock Convex backend for fast automated testing
- [Testing Local Backend](/testing/convex-backend.md): Test functions using the local open-source backend
- [Continuous Integration](/testing/ci.md): Set up continuous integration testing

## Production

- [Deploying Your App to Production](/production.md): Tips for building safe and reliable production apps
- [Environment Variables](/production/environment-variables.md): Store and access environment variables
- [Hosting and Deployment](/production/hosting.md): Share your Convex backend and web app
- [Using Convex with Vercel](/production/hosting/vercel.md): Host your frontend on Vercel
- [Using Convex with Netlify](/production/hosting/netlify.md): Host your frontend on Netlify
- [Preview Deployments](/production/hosting/preview-deployments.md): Use Convex with preview deployments
- [Custom Domains & Hosting](/production/hosting/custom.md): Serve requests from custom domains
- [Integrations](/production/integrations.md): Integrate Convex with third party services
- [Log Streams](/production/integrations/log-streams.md): Configure logging integrations
- [Exception Reporting](/production/integrations/exception-reporting.md): Configure exception reporting
- [Streaming Data in and out of Convex](/production/integrations/streaming-import-export.md): Streaming data
- [Project Configuration](/production/project-configuration.md): Configure your Convex project
- [Multiple Repositories](/production/multiple-repos.md): Use Convex in multiple repositories
- [Pausing a Deployment](/production/pause-deployment.md): Temporarily disable a deployment
- [Limits](/production/state/limits.md): Platform limits
- [Status and Guarantees](/production/state.md): Production guarantees and availability

## Self Hosting

- [Self Hosting](/self-hosting.md): Self Hosting Convex Projects

## CLI

- [CLI](/cli.md): Command-line interface for managing Convex projects
- [Deploy keys](/cli/deploy-key-types.md): Use deploy keys for authentication in production
- [Local Deployments](/cli/local-deployments.md): Develop with local deployments
- [Agent Mode](/cli/agent-mode.md): Configure anonymous development mode for cloud agents

## Client Libraries

### React
- [Convex React](/client/react.md): React client library for Convex
- [Deployment URLs](/client/react/deployment-urls.md): Configuring your project
- [Optimistic Updates](/client/react/optimistic-updates.md): Make your React app more responsive

### Next.js
- [Next.js](/client/nextjs/app-router.md): How Convex works in a Next.js app
- [Server Rendering](/client/nextjs/app-router/server-rendering.md): Server-side rendering with Convex
- [Pages Router](/client/nextjs/pages-router.md): Using Convex with Next.js Pages Router

### Other Frameworks
- [React Native](/client/react-native.md): How Convex works in a React Native app
- [Svelte](/client/svelte.md): Reactive Svelte client library
- [Vue](/client/vue.md): Community-maintained Vue integration
- [Nuxt](/client/vue/nuxt.md): Nuxt integration
- [TanStack Start](/client/tanstack/tanstack-start.md): How Convex works with TanStack Start
- [TanStack Query](/client/tanstack/tanstack-query.md): Integrate Convex with TanStack Query

### JavaScript/Node
- [JavaScript Clients](/client/javascript.md): JavaScript clients for Node.js and browser
- [Node.js](/client/javascript/node.md): Use Convex HTTP and subscription clients in Node.js
- [Bun](/client/javascript/bun.md): Use Convex clients with Bun
- [Script Tag](/client/javascript/script-tag.md): Use Convex directly in HTML

### Native
- [Android Kotlin](/client/android.md): Android Kotlin client library
- [iOS & macOS Swift](/client/swift.md): Swift client library
- [Rust](/client/rust.md): Rust client library
- [Python](/client/python.md): Python client library

### Other
- [OpenAPI & Other Languages](/client/open-api.md): Connecting from other languages

## Dashboard

- [Dashboard](/dashboard.md): Learn how to use the Convex dashboard
- [Deployments](/dashboard/deployments.md): Understand Convex deployments
- [Data](/dashboard/deployments/data.md): View, edit, and manage database tables
- [Functions](/dashboard/deployments/functions.md): Run, test, and monitor functions
- [Logs](/dashboard/deployments/logs.md): View real-time function logs
- [File Storage](/dashboard/deployments/file-storage.md): Manage stored files
- [Schedules](/dashboard/deployments/schedules.md): Monitor scheduled functions and cron jobs
- [Health](/dashboard/deployments/health.md): Monitor deployment health
- [History](/dashboard/deployments/history.md): View audit log of events
- [Settings](/dashboard/deployments/deployment-settings.md): Configure deployment settings
- [Projects](/dashboard/projects.md): Create and manage projects
- [Teams](/dashboard/teams.md): Manage team settings and members

## Errors and ESLint

- [Errors and Warnings](/error.md): Understand specific errors thrown by Convex
- [ESLint rules](/eslint.md): ESLint rules for Convex

## Tutorial

- [Chat App Tutorial](/tutorial.md): Build a real-time chat application
- [Calling External Services](/tutorial/actions.md): Extend your chat app with external APIs
- [Scaling Your App](/tutorial/scale.md): Learn how to scale your Convex application

## Generated Code

- [Generated Code](/generated-api.md): Auto-generated JavaScript and TypeScript code
- [api.js](/generated-api/api.md): Generated API references for your functions
- [dataModel.d.ts](/generated-api/data-model.md): Generated TypeScript types for your schema
- [server.js](/generated-api/server.md): Generated utilities for implementing functions

## HTTP API

- [Convex HTTP API](/http-api.md): Connecting to Convex directly with HTTP

## Platform APIs

- [Platform APIs](/platform-apis.md): Convex Platform APIs (Beta)
- [Deployment API](/deployment-api.md): Admin API for interacting with deployments
- [Management API](/management-api.md): Creating and managing deployments by API
- [Streaming Export](/streaming-export-api.md): Streaming data out of Convex
- [Streaming Import](/streaming-import-api.md): Streaming data into Convex
