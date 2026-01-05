# Example: Draft from Scratchpad

Complete example of the documentation drafting pipeline.

---

## Input: Scratchpad

```markdown
# Authentication API

## Overview
The auth API handles user authentication and session management.

## Endpoints

### POST /auth/login
Authenticates a user and returns a session token.

Request:
```json
{
  "email": "user@example.com",
  "password": "secret"
}
```

Response:
```json
{
  "token": "eyJ...",
  "expires_at": "2025-01-05T10:00:00Z"
}
```

### POST /auth/logout
Invalidates the current session.

### GET /auth/me
Returns the current user's profile.

## Notes
- Tokens expire after 24 hours
- Rate limited to 5 attempts per minute
- Need to document error codes
```

---

## Execution

### Phase 1: INIT

```bash
draft-orchestrator --input scratchpad.md --output draft.mdx
```

**Actions**:
1. Resolve workspace paths
2. Create manifest
3. Verify input file exists

### Phase 2: EXTRACT

**Script**: `extract_code_blocks.py`

**Output**: `code_manifest.json`
```json
{
  "blocks": [
    {
      "id": "block_001",
      "language": "json",
      "content": "{\n  \"email\": \"user@example.com\",\n  \"password\": \"secret\"\n}",
      "context": "POST /auth/login request"
    },
    {
      "id": "block_002",
      "language": "json",
      "content": "{\n  \"token\": \"eyJ...\",\n  \"expires_at\": \"2025-01-05T10:00:00Z\"\n}",
      "context": "POST /auth/login response"
    }
  ]
}
```

### Phase 3: ANALYZE

**Agent**: docs-detective (non-blocking)

```javascript
Task({
  subagent_type: "docs-detective",
  prompt: `
    Analyze: scratchpad.md
    Find gaps, missing context, undefined terms.
    Output: questions.md
  `
})
```

**Output**: `questions.md`
```markdown
## Questions

1. What error codes can /auth/login return?
2. What's the rate limit response format?
3. Is there a refresh token mechanism?
4. What authentication method is used (Basic, Bearer)?
```

**Agent**: docs-architect (blocking)

```javascript
Task({
  subagent_type: "docs-architect",
  prompt: `
    Create blueprint for: scratchpad.md
    Code manifest: code_manifest.json
    Questions: questions.md
    Output: blueprint.md
  `
})
```

**Output**: `blueprint.md`
```markdown
## Blueprint

**Document Type**: API Reference

### Structure

1. **Overview** (required)
   - What the API does
   - Authentication method

2. **Authentication** (required)
   - How to authenticate
   - Token format

3. **Endpoints** (required)
   - POST /auth/login
     - Code: block_001 (request)
     - Code: block_002 (response)
   - POST /auth/logout
   - GET /auth/me

4. **Error Handling** (required)
   - <!-- INPUT: What error codes? -->

5. **Rate Limiting** (required)
   - Limits
   - <!-- INPUT: Response format? -->

### Code Block Placement
- block_001: Endpoints > POST /auth/login > Request
- block_002: Endpoints > POST /auth/login > Response
```

### Phase 4: DRAFT

**Agent**: docs-writer

```javascript
Task({
  subagent_type: "docs-writer",
  prompt: `
    Write draft from:
    - Blueprint: blueprint.md
    - Scratchpad: scratchpad.md
    - Code manifest: ${JSON.stringify(codeManifest)}

    Output: draft.mdx

    Rules:
    - Use <!-- CODE:block_id --> placeholders
    - Use <!-- INPUT: question --> for gaps
    - Follow blueprint structure exactly
  `
})
```

**Output**: `draft.mdx`
```mdx
---
title: Authentication API
description: User authentication and session management
---

# Authentication API

## Overview

The Authentication API handles user authentication and session management for your application.

## Authentication

All authenticated endpoints require a Bearer token in the Authorization header:

```
Authorization: Bearer <token>
```

## Endpoints

### POST /auth/login

Authenticates a user and returns a session token.

**Request**

<!-- CODE:block_001 -->

**Response**

<!-- CODE:block_002 -->

### POST /auth/logout

Invalidates the current session. Requires authentication.

### GET /auth/me

Returns the current user's profile. Requires authentication.

## Error Handling

<!-- INPUT: What error codes can the auth endpoints return? -->

## Rate Limiting

The login endpoint is rate limited to 5 attempts per minute per IP address.

<!-- INPUT: What is the rate limit response format? -->
```

### Phase 5: INJECT

**Script**: `inject_code_blocks.py`

**Output**: `draft.mdx` (updated)
```mdx
---
title: Authentication API
description: User authentication and session management
---

# Authentication API

## Overview

The Authentication API handles user authentication and session management for your application.

## Authentication

All authenticated endpoints require a Bearer token in the Authorization header:

```
Authorization: Bearer <token>
```

## Endpoints

### POST /auth/login

Authenticates a user and returns a session token.

**Request**

```json
{
  "email": "user@example.com",
  "password": "secret"
}
```

**Response**

```json
{
  "token": "eyJ...",
  "expires_at": "2025-01-05T10:00:00Z"
}
```

### POST /auth/logout

Invalidates the current session. Requires authentication.

### GET /auth/me

Returns the current user's profile. Requires authentication.

## Error Handling

<!-- INPUT: What error codes can the auth endpoints return? -->

## Rate Limiting

The login endpoint is rate limited to 5 attempts per minute per IP address.

<!-- INPUT: What is the rate limit response format? -->
```

### Phase 6: VALIDATE

**Script**: `validate_draft.py`

**Output**: `validation.json`
```json
{
  "status": "WARNINGS",
  "blocking_errors": [],
  "warnings": [
    "2 INPUT markers remain - requires user input",
    "Error Handling section is placeholder only"
  ],
  "recoverable": false
}
```

### Phase 7: REPORT

```yaml
summary: Draft created with 2 input markers requiring user attention
artifacts:
  draft: /path/to/draft.mdx
  blueprint: /path/to/blueprint.md
  questions: /path/to/questions.md
validation:
  status: WARNINGS
  warnings:
    - "2 INPUT markers remain"
    - "Error Handling section incomplete"
next_actions:
  - "Fill in error codes for Error Handling section"
  - "Add rate limit response format"
```

---

## Notes

- Code blocks are extracted and re-injected to preserve formatting
- INPUT markers flag gaps that need human input
- The detective runs non-blocking (can inform architect but doesn't block)
- Blueprint drives the structure - writer follows it exactly
