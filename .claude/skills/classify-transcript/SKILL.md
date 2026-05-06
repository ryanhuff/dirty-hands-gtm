# Classify Transcript

Examine a transcript (title and/or content) and determine whether it is a **sales call** worth syncing to the customer intelligence pipeline. This skill is used by `/sync-transcripts` to filter out non-sales calls automatically.

## Trigger

This skill is NOT user-invocable. It is called internally by other skills (primarily `/sync-transcripts`) that need to classify a transcript before deciding whether to download or process it.

## Classification: Sales Call vs. Non-Sales Call

A transcript qualifies as a **sales call** if it involves a **prospect or customer evaluating, purchasing, onboarding, or using Quotivity**. The call must include at least one external participant who represents a company that is or could become a Quotivity customer.

### Qualifies as a sales call

- **Discovery / intro calls**: Quotivity team meeting a prospect for the first time
- **Demo calls**: Showing product capabilities to a prospect
- **Reverse demos**: Prospect walking Quotivity through their current process
- **Evaluation / POC calls**: Prospect testing or reviewing the product
- **Requirements reviews**: Scoping what a prospect needs
- **Follow-up calls**: Continuing a conversation with a prospect or customer
- **Proposal / onboarding calls**: Discussing pricing, contracts, or implementation
- **Pre-demo conversations**: Prep calls that include an external prospect

### Does NOT qualify as a sales call

- **Internal team meetings**: Calls between Quotivity team members only (e.g., "Bryan / Daniel Reverse Demo Weekly Update")
- **1:1 partner calls**: Calls between a Quotivity team member and a single partner, agency, or consultant where no prospect is present and the conversation is about internal strategy, not a specific deal with a customer on the call (e.g., "Ryan / Daniel", "Kathleen Swift and Daniel Gomez" if Kathleen is a partner)
- **Meeting code artifacts**: Auto-generated meeting stubs with no real transcript content (e.g., "anf-vpvk-tdu (2026-03-16 12:00 GMT-4)")
- **Empty or near-empty transcripts**: Files with less than 1KB of content or only a few lines of "hello/goodbye"

## Classification Logic

### Step 1: Title-based classification (fast path)

Many transcripts can be classified from the title alone without reading content:

**Auto-SKIP patterns** (definitely not sales calls):
- Title matches `Bryan / Daniel *` or `Ryan / Daniel *` — internal weekly syncs
- Title is a meeting code pattern: 3 random letters, hyphen, 4 random letters, hyphen, 3 random letters (e.g., `anf-vpvk-tdu`)
- File size is 1024 bytes or less (Google Drive minimum; indicates an empty/stub transcript)

**REQUIRES CONTENT REVIEW** (title alone is not enough):
- Title is only two personal names joined by "and" with no company context (e.g., "Casey Thurman and Daniel Gomez", "William taylor and Daniel Gomez"). These could be partner strategy calls OR prospect calls. Must read content to classify. See Step 2.

**Auto-INCLUDE patterns** (definitely sales calls):
- Title contains `{Company} and Quotivity {type}` where type is: Introduction, Demo, Reverse Demo, Follow Up, Evaluation, CPQ Evaluation, Requirements Review, Proposal, Onboarding, Sync, Template Support
- Title contains `Quotivity Introduction for {Person} @ {Company}`
- Title contains `Quotivity Reverse Demo with/w/ {Company}`
- Title contains `Quotivity/` followed by a company name (e.g., "Quotivity/Dalux POC Goals")
- Title contains `{Company} pre demo conversation` or `{Company} Pre Demo meeting`

**Known Quotivity team members** (if ALL participants are from this list, it's internal):
- Bryan Roy
- Daniel Gomez
- Ryan Huff

**Note on partner calls**: Partner/agency 1:1 calls cannot be reliably identified from the title alone. A title like "Kathleen Swift and Daniel Gomez" could be a partner strategy session or a prospect intro. These always require content review (Step 2).

### Step 2: Content-based classification (when title is ambiguous)

If the title doesn't clearly match an auto-skip or auto-include pattern, read the transcript content and classify based on what's actually being discussed. This step is **required** for person-name-only titles (e.g., "Kathleen Swift and Daniel Gomez") since these could be either partner strategy calls or prospect calls.

Read the first 100 lines of the transcript content and check:

1. **Attendees list**: If an attendees section exists, check whether any attendee is from an external company (not Quotivity, not a known partner/agency)
2. **Partner 1:1 signals** (classify as `not_sales_call` with reason "partner 1:1"):
   - Only 2 participants talking, one is Quotivity team
   - The other person is a partner/agency rep coordinating with Quotivity about a deal they are running — the prospect is NOT on the call
   - Any prospect or customer is discussed in the **third person** ("they", "these guys", "the client", "a company called X") rather than being a participant
   - Partner-specific language: "get me some pricing", "your guys' tool", "we demoed your tool to them", "part of the solution we're presenting", "partner program", "can we handle the onboarding"
   - Discussion of partner logistics: pricing to include in the partner's proposal, who handles onboarding, co-selling coordination, referral terms
   - Also covers broader partner strategy: pipeline reviews, go-to-market coordination, content strategy, product roadmap discussions
3. **Sales call signals** (classify as `sales_call`):
   - A prospect or customer is present and speaking on the call
   - Product evaluation language directed at/from a participant: "your trial", "let me show you", "we're evaluating", "our requirements", "pricing for us"
   - The non-Quotivity participant describes their own company's current process, pain points, or needs
   - Discussion of specific deal terms, implementation timelines, or onboarding for the participant's company
4. **When both signals are absent**: Classify as `uncertain`

### Step 3: Return classification

Return one of:
- `sales_call` — include in sync
- `not_sales_call` — skip, with a brief reason (e.g., "internal weekly sync", "partner 1:1", "empty transcript", "meeting code stub")
- `uncertain` — could not determine; include the transcript and flag it for the user to review

When uncertain, **err on the side of including** the transcript. It's better to sync a non-sales call than to miss a sales call.

## Output Format

When called by another skill, return a structured classification:

```
title: {original Drive file title}
classification: sales_call | not_sales_call | uncertain
reason: {brief explanation}
company_slug: {extracted company slug, if sales_call}
```
