# Sprint Planning — Agent Prompt

Read the following sprint planning meeting transcript and set up Sprint 30 in InteliG.

## Transcript
sources/meetings/payments-sprint-30-planning.md

## InteliG API
Base URL: https://app.intelig.ai/api/v1/intelig
Authorization: ApiKey $INTELIG_API_KEY

## What to do
1. Read the transcript — extract sprint dates, goals, owners, and key results
2. GET /strategy/sprints — check for overlapping dates
3. POST /strategy/sprints — create Sprint 30 (March 8–22, 2026)
4. PUT /strategy/sprints/{id}/initiatives — link the 3 initiatives discussed
5. PUT /strategy/sprints/{id}/goals — set goals with owners and KRs extracted from the meeting
6. The action items were already extracted by InteliG from the transcript — cross-reference them
7. POST /strategy/sprints/{id}/intelligence/generate — generate the AI sprint analysis
8. Report what you created

Don't ask me anything — the meeting already happened. Execute.
