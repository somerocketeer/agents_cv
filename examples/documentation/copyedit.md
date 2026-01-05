# Example: Copyedit Workflow

Example of the single-pass copyedit orchestrator.

---

## Input: Existing Documentation

```mdx
---
title: Getting Started
---

# Getting Started

## Introduction

In order to get started with our API, you will need to first obtain an API key. The API key can be obtained by signing up for an account on our website.

## Installation

The SDK can be installed using npm. It is recommended that you use the latest version of the SDK for the best experience.

```bash
npm install our-sdk
```

## Configuration

Configuration of the SDK is done by passing options to the constructor. The options that are available are listed below.

```javascript
const sdk = new SDK({
  apiKey: process.env.API_KEY,
  timeout: 30000
});
```

## Making Requests

Requests can be made by calling methods on the SDK instance. The response will be returned as a promise that resolves to the result.

```javascript
const result = await sdk.getData();
```

It should be noted that all requests are rate limited.
```

---

## Execution

```bash
editor-orchestrator --target docs/getting-started.mdx
```

### Analysis

**docs-copyedit** identifies issues:

| Line | Issue | Type |
|------|-------|------|
| 7 | "In order to" | Filler |
| 7 | "you will need to first" | Hedge stacking |
| 7 | Passive: "can be obtained" | Voice |
| 11 | "It is recommended that you" | Hedge |
| 11 | "for the best experience" | Filler |
| 15 | "Configuration of the SDK is done by" | Passive |
| 15 | "The options that are available are" | Wordy |
| 25 | "Requests can be made by calling" | Passive |
| 25 | "will be returned as a promise that resolves" | Wordy |
| 29 | "It should be noted that" | Filler |

### Fixes Applied

**Original**:
> In order to get started with our API, you will need to first obtain an API key. The API key can be obtained by signing up for an account on our website.

**Fixed**:
> To get started, obtain an API key by signing up on our website.

---

**Original**:
> It is recommended that you use the latest version of the SDK for the best experience.

**Fixed**:
> Use the latest version of the SDK.

---

**Original**:
> Configuration of the SDK is done by passing options to the constructor. The options that are available are listed below.

**Fixed**:
> Configure the SDK by passing options to the constructor:

---

**Original**:
> Requests can be made by calling methods on the SDK instance. The response will be returned as a promise that resolves to the result.

**Fixed**:
> Call methods on the SDK instance. Methods return promises.

---

**Original**:
> It should be noted that all requests are rate limited.

**Fixed**:
> All requests are rate limited.

---

## Output

```mdx
---
title: Getting Started
---

# Getting Started

## Introduction

To get started, obtain an API key by signing up on our website.

## Installation

Install the SDK using npm:

```bash
npm install our-sdk
```

Use the latest version of the SDK.

## Configuration

Configure the SDK by passing options to the constructor:

```javascript
const sdk = new SDK({
  apiKey: process.env.API_KEY,
  timeout: 30000
});
```

## Making Requests

Call methods on the SDK instance. Methods return promises.

```javascript
const result = await sdk.getData();
```

All requests are rate limited.
```

---

## Checklist

The copyedit pass checks:

- [x] Active voice (>90%)
- [x] No "in order to"
- [x] No "it should be noted"
- [x] No "it is recommended"
- [x] Sentences under 25 words
- [x] No hedge stacking
- [x] Imperative mood for instructions
- [x] Code blocks have language tags
