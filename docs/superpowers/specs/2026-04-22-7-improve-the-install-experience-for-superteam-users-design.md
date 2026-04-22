# Design: Improve the install experience for Superteam users [#7](https://github.com/patinaproject/superteam/issues/7)

## Summary

Improve the install experience for `superteam` by treating the repository landing page as the primary onboarding surface for new users. The design focuses on helping a first-time visitor quickly understand what Superteam does, what problem it solves, how it works with agent teams or subagents, and how the workflow continues cleanly across handoffs.

The design keeps the issue outcome-focused rather than prescribing a README-only rewrite. The repository landing page is the primary surface to improve first, but the broader install experience should be evaluated in terms of whether the user can move from discovery to first use without losing clarity.

## Goals

- Make the repository landing page explain what Superteam does before it explains repository structure or install surfaces
- Help a first-time visitor understand the problem Superteam solves
- Make the continuity and handoff model visible early in the docs experience
- Clarify that Superteam works with agent teams or subagents
- Avoid implying that one runtime is better than the other without evidence
- Connect installation to a clear first-use path instead of ending at setup

## Non-Goals

- Reworking the underlying `superteam` skill behavior
- Expanding the scope into contributor-focused documentation improvements
- Defining final README copy line-by-line in the design
- Committing to a README-only solution before evaluating the broader install journey

## User And Problem

The primary user is someone landing on the repository to evaluate or install Superteam. That user should not need to understand the repo structure first. They need to understand the product first.

The main problem is that the current landing experience explains install surfaces, but it does not yet explain the product value clearly enough. A new user can discover that there are multiple install surfaces, but still leave without understanding:

- what Superteam actually does
- why it exists
- how it differs from a simpler ad hoc subagent workflow
- why continuity across agent handoffs is a core part of the system
- what to do immediately after installation

## Recommended Approach

Use a problem-first onboarding model for the install experience, with the repository landing page as the lead surface.

The landing page should introduce Superteam as a workflow for orchestrating teams of agents with Superpowers. It should establish the user problem before it explains installation mechanics. The problem statement should set up the continuity and handoff story, because the skill itself is built around durable artifacts, stage ownership, and resumable progress across a single issue workflow.

The preferred headline direction is:

`Orchestrate teams of agents with Superpowers`

That positioning should be followed immediately by copy that explains the actual operating model. Superteam is not just generic agent automation. It is a structured issue workflow that lets teams of agents pick up, hand off, and continue work without losing context.

## Approach Options Considered

### Option 1: Problem-first landing page with a single flagship workflow

Pros:
- Aligns the repo landing page with the actual `superteam` skill behavior
- Gives first-time users a clear mental model before setup details
- Makes continuity and team handoff visible early
- Leaves room to explain compatibility without implying a runtime preference

Cons:
- Requires restructuring the current landing-page narrative
- May expose supporting docs gaps that also need follow-up work

### Option 2: Install-first landing page with stronger product copy

Pros:
- Smaller documentation change
- Easier to implement incrementally

Cons:
- Keeps setup mechanics ahead of understanding
- Does not solve the core problem that users may still not understand the product after landing

### Option 3: Docs hub approach with a short landing page and deeper subpages

Pros:
- Scales well if the docs set grows
- Makes it easier to separate install, workflow, and contributor topics

Cons:
- Weakens the repository landing page as the primary onboarding surface
- Adds indirection before the user understands what Superteam does

Selected option: Option 1.

## Information Architecture

### Landing Page Role

The repository landing page should act as product onboarding for the install experience. It should not begin as a repository structure reference.

The top of the page should help a new user answer these questions in order:

1. What is Superteam?
2. What problem does it solve?
3. How does it work at a high level?
4. Does it work with agent teams or subagents?
5. How do I install it and what do I do next?

### Workflow Presentation

The docs should center on one flagship workflow rather than several entry points. That workflow should show how a GitHub issue moves through:

- brainstorm
- plan
- execute
- pre-push review
- finish
- comment handling

This matters because the skill itself is explicitly stage-based. The workflow is not just conceptual marketing. It is the actual operating model enforced by the skill.

### Runtime Positioning

The landing page should state clearly that Superteam works with agent teams or subagents. It should avoid saying one runtime works better than the other unless that claim has been validated.

Compatibility language should support the main story about structured issue workflows and continuity, rather than becoming the primary claim.

## Continuity And Handoff Model

The design should explain continuity as the core differentiator in the install experience.

From the skill itself, continuity comes from:

- explicit stages
- owned artifacts for each stage
- approval gates between stages
- verification and reporting requirements
- finish-stage publication and review follow-through

The landing page should help users understand that Superteam is designed so another agent, or a human, can resume work without reconstructing intent from scratch. The docs do not need to describe every rule in detail, but they should make the handoff model legible.

## Install Experience Expectations

Installation guidance should follow understanding rather than lead it. The broader install experience should be evaluated against whether the user can move cleanly through these steps:

1. Understand the problem and value
2. Understand the flagship workflow
3. Understand runtime compatibility
4. Choose the relevant install surface
5. Understand the first step after installation

If installation instructions exist without that narrative, the user experience remains incomplete even if the commands themselves are correct.

### Claude Code Setup Guidance

Within the user-facing install guidance, the Claude Code path should remain complete and usable on its own. It should not require Agent Teams to make sense.

That same Claude Code section should also include a short optional subsection for enabling Agent Teams. The purpose of that subsection is to help users who want a team-oriented runtime without making that setup feel mandatory.

The optional subsection should:

- give quick setup guidance for Agent Teams
- keep the default setup path intact for regular single-agent or subagent use
- include one short factual explanation of the difference

The difference should be explained briefly: Agent Teams provides a team-oriented runtime where multiple agents can coordinate through the staged workflow, while the regular setup runs the same workflow with a single agent or subagents.

## Inspiration And Constraints

The design may borrow patterns from strong open source landing pages and workflow-first repositories, including:

- direct problem and value framing near the top
- early workflow explanation
- concise diagrams to communicate system behavior quickly
- install instructions that follow product understanding instead of preceding it

Those references should inform the shape of the docs, but the design should stay grounded in the actual `superteam` skill behavior rather than imitating another repository's structure.

## Testing And Verification

Validation for this work should focus on clarity and narrative flow rather than code execution.

- review the repository landing page for whether the product purpose is understandable in a short scan
- verify that compatibility with agent teams or subagents is stated clearly
- verify that the landing page does not imply an unvalidated runtime preference
- verify that the flagship workflow and continuity story are understandable before installation details
- verify that install guidance leads into an explicit first-use next step
- verify that the Claude Code install path includes an optional Agent Teams subsection with a brief explanation of how it differs from regular setup

## Acceptance Criteria

- AC-7-1: The user-facing install experience leads with what Superteam does and the problem it solves before explaining install surfaces
- AC-7-2: The repository landing page presents one clear flagship workflow for how Superteam operates across an issue
- AC-7-3: The landing experience explains that Superteam works with agent teams or subagents
- AC-7-4: The landing experience avoids implying an unvalidated runtime preference
- AC-7-5: The continuity and handoff model is understandable from the initial docs experience
- AC-7-6: Installation guidance connects to a clear first-use next step instead of ending at setup
- AC-7-7: The Claude Code install guidance includes an optional Agent Teams subsection with a brief explanation of how Agent Teams differs from regular setup

## Implementation Notes

- Keep the issue focused on the install experience as a whole, not just a README rewrite
- Prefer capability and workflow language that reflects orchestration and continuity over generic automation language
- Keep contributor-facing structural details below the initial onboarding story
