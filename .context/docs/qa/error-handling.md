---
slug: error-handling
category: operations
status: filled
generatedAt: 2026-03-18T21:32:54.231Z
relevantFiles:
  - src/workflow/errors.ts
  - src/services/shared/types.ts
  - src/services/passthrough/protocol.ts
---

# How are errors handled?

## Overview

Error handling in this project follows three patterns depending on the context: workflow errors (custom error classes), operation result errors (structured result objects), and protocol errors (JSON error responses).

## 1. Workflow errors (`src/workflow/errors.ts`)

The PREVC workflow engine uses a class hierarchy for typed errors:

- **`WorkflowError`** -- Base error class for all workflow-related failures.
- **`WorkflowGateError`** -- Thrown when a workflow gate blocks a phase transition. Carries the `transition` (from/to phases), the `gate` type, and a human-readable `hint` for resolution.
- **`NoPlanToApproveError`** -- Thrown when attempting to approve a plan that has not been linked.
- **`NoWorkflowError`** -- Thrown when a workflow operation is attempted but no workflow exists.

All workflow errors extend the native `Error` class and set a descriptive `name` property for easy identification in catch blocks.

## 2. Operation result errors (`src/services/shared/types.ts`)

File-based operations (import, export, sync) use an `OperationResult` pattern instead of throwing:

```typescript
interface OperationResult {
  filesCreated: number;
  filesSkipped: number;
  filesFailed: number;
  errors: OperationError[];
}

interface OperationError {
  file: string;
  error: string;
}
```

The `addError()` helper increments `filesFailed` and appends an `OperationError` entry. Multiple results can be merged with `mergeResults()`. This pattern allows batch operations to continue processing after individual file failures and report all errors at the end.

## 3. Passthrough protocol errors (`src/services/passthrough/protocol.ts`)

The JSON-over-stdin/stdout passthrough protocol returns structured error responses with typed error codes:

```typescript
interface ErrorResponse {
  id: string;
  success: false;
  error: {
    code: string;    // One of ErrorCodes
    message: string;
    details?: unknown;
  };
}
```

Defined error codes:

| Code | Meaning |
| --- | --- |
| `PARSE_ERROR` | Malformed JSON input |
| `INVALID_REQUEST` | Request fails Zod schema validation |
| `METHOD_NOT_FOUND` | Unknown method in request |
| `TOOL_NOT_FOUND` | Requested tool does not exist |
| `AGENT_NOT_FOUND` | Requested agent does not exist |
| `EXECUTION_ERROR` | Runtime error during tool/agent execution |
| `VALIDATION_ERROR` | Parameter validation failure |

The `createErrorResponse()` factory function produces correctly shaped error responses.

## 4. CLI-level error handling

At the CLI level (`src/index.ts`), errors from services are caught and displayed to the user through the `CLIInterface` (`ui`) which formats output with chalk colors. Unhandled promise rejections and uncaught exceptions will cause the process to exit with a non-zero code.

## General conventions

- **Do not throw for expected batch failures** -- Use `OperationResult` to accumulate errors when processing multiple files.
- **Throw custom error classes for workflow violations** -- Callers can use `instanceof` to handle specific workflow error types.
- **Return structured error responses in protocol mode** -- Never throw across the JSON protocol boundary; always respond with an `ErrorResponse`.
- **Include actionable hints** -- `WorkflowGateError` includes a `hint` field to guide users toward resolution.
