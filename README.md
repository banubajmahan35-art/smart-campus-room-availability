# Smart Campus Room Availability Checker

A small, responsive React app that checks whether **Room A-101** is free for a
given date and time. All data is fictional — there is no login, database, or
real API key involved.

## Availability rules

The availability checker is the **single source of truth** for whether the
room is free. The explanation feature never changes or overrides that result.
The room has two fictional existing bookings:

- **10:00 – 11:00**
- **14:00 – 15:30**

The checker returns one of these reason codes:

| Code | Meaning |
| --- | --- |
| `AVAILABLE` | The requested slot fits between existing bookings |
| `OVERLAP` | The requested slot overlaps an existing booking |
| `INVALID_TIME` | End time is earlier than or equal to the start time |
| `PAST_TIME` | The date or start time is in the past |
| `EMPTY_INPUT` | One or more fields were left empty |

Touching slots are allowed (e.g. 09:00–10:00 is fine because the 10:00 booking
starts exactly when the slot ends).

## Project layout

```
src/
  lib/availability.ts        # availability rules (pure functions, no UI)
  lib/availability.test.ts   # automated tests for those rules
  lib/explainer.ts           # mock LLM adapter for the explanation feature
  lib/explainer.test.ts      # automated tests for the explanation feature
  App.tsx                    # the user interface
```

The availability-checking logic lives in its own module and is fully
deterministic — the UI simply calls `checkAvailability()` and displays the
result.

## Explain availability (mock LLM)

After a successful availability check, a separate **“Explain availability”**
button (visually distinct from the primary button) sends a structured payload
to a **mock LLM adapter**:

```json
{
  "roomLabel": "Room A-101",
  "date": "2026-06-20",
  "startTime": "12:00",
  "endTime": "13:00",
  "isAvailable": true,
  "reasonCode": "AVAILABLE"
}
```

The adapter returns `{ "explanation": "..." }` — at most two sentences,
using only the supplied facts, and always agreeing with the confirmed
availability result. No API key, network call, or real AI model is used.

Safety properties:

- **Input validation** — empty or oversized fields are rejected before any
  explanation is produced.
- **Output validation** — the adapter's response is treated as untrusted
  data: it must be a non-empty string of at most 300 characters and no more
  than two sentences, or it is rejected.
- **Timeout / API failure simulation** — the adapter can be run in
  `"timeout"` or `"failure"` modes for testing. In those cases a safe,
  deterministic fallback explanation is shown, based only on facts already
  confirmed by the availability checker.
- **Separation** — the explanation appears in its own section labelled
  *“AI-generated explanation”*, clearly separate from the confirmed
  availability status, which it can never change.

## Getting started

```bash
npm install
```

### Run the dev server

```bash
npm run dev
```

Then open the printed local URL in your browser.

### Run the tests

```bash
npm test
```

This runs two suites:

- **availability tests** — the deterministic booking rules (available,
  overlap, invalid times, past times, empty input);
- **explainer tests** — the explanation feature (available explanation,
  overlap explanation, blocked/invalid results, timeout fallback, simulated
  API failure, malformed adapter response, and confirmation that an
  explanation cannot change availability).

### Lint the code

```bash
npm run lint
```

### Type-check

```bash
npm run typecheck
```

### Build for production

```bash
npm run build
```

The optimized output is written to the `dist/` folder. Preview it locally with
`npm run preview`.
