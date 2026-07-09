You are a world-class Principal Product Engineer, Game UI Designer, DevSecOps Architect, Svelte/TypeScript Engineer, Three.js/Threlte 3D Engineer, Data Modelling Architect, and LLM Prompt Refinement Specialist.

Your task is to refine, enhance, architect, and then build out a browser-based gamified DevSecOps/SDLC racing leaderboard called:

DevSecOps Grand Prix

This application visualises progress across hundreds of software repositories as a live arcade-style 3D race. Teams, platforms, squads, or repo groups are represented as racing cars moving around a full 3D track. The race progress is driven by verified SDLC and DevSecOps maturity metrics.

The initial prototype must be driven entirely from CSV files so the concept can be proven without needing live integrations yet.

You must think deeply, challenge weak assumptions, fill all architectural gaps, and produce an implementation that is clean, modular, extensible, testable, and production-upgradable.

Do not produce a shallow demo. Produce the strongest possible MVP foundation that can later become a real internal engineering product.

---

# 1. Product Vision

Build a modern browser app that turns DevSecOps and SDLC remediation progress across many repositories into a live 3D racing experience.

The user wants something visually inspired by:

- arcade racing dashboards
- full 3D browser-based race tracks
- live esports-style leaderboards
- Daytona-style race spectacle
- Fortune Cup-style spectator race presentation

Do not copy any existing game, trademarked artwork, branding, logos, or proprietary visual assets. The design must be original.

The product should feel like:

- a live engineering championship
- a security maturity race
- a repo fleet remediation dashboard
- a stakeholder-friendly visualisation
- an operational tool, not just a toy

The key metaphor:

- Each team/lane is a race car.
- Each repo fleet is a racing lane/group.
- Each SDLC/DevSecOps control group is a checkpoint or track sector.
- Verified progress moves the car forward.
- Failed controls put the car into pit lane.
- Regressions move the car backwards or mark the car blocked.
- Overtakes appear as live race events.
- The leaderboard ranks teams by weighted verified maturity progress.

---

# 2. Critical Product Principle

The game must reward verified risk reduction, not ticket spam.

The scoring system must not simply count completed tasks. It must be based on machine-verifiable control state.

Good scoring:

- repo ownership verified
- CODEOWNERS present
- branch protection enabled
- CI checks required and passing
- SAST enabled and current
- dependency scanning enabled
- no critical/high vulnerabilities
- secret scanning enabled
- SBOM generated
- container scanning passing
- IaC scanning enabled
- production readiness evidence present

Bad scoring:

- number of Jira tickets closed
- self-attested completion
- raw number of repos touched
- easy low-risk repo farming
- temporary compliance with no freshness expiry

The prototype can simulate this from CSV, but the architecture must make it clear how real evidence sources will replace the CSV later.

---

# 3. First Build Constraint

For now, assume all data is stored in CSV files.

The app must read CSV from the local/public app directory.

The CSV should group repositories into lanes and define the metric groups used to calculate progress.

The initial system must support:

- multiple lanes/teams
- multiple repos per lane
- multiple controls per repo
- weighted scoring
- repo criticality weighting
- freshness decay
- failed/blocking controls
- derived leaderboard ranking
- derived car position
- derived pit lane status
- derived event feed

The renderer must not care whether the data came from CSV, API, WebSocket, GitHub, GitLab, Jira, CI/CD, or scanner tools.

Use this strict boundary:

CSV/data source
  -> typed domain model
  -> scoring engine
  -> race state
  -> renderer/UI

---

# 4. Preferred Technical Stack

Use:

- Svelte
- Vite
- TypeScript
- Three.js
- Threlte if appropriate for Svelte-native Three.js usage
- Papa Parse or equivalent robust CSV parser
- Zod or equivalent validation library
- CSS/Tailwind if useful, but avoid overcomplicating styling
- No backend required for the first prototype unless you strongly justify it

The app should run locally with:

npm install
npm run dev

Do not use Docker for the initial prototype unless explicitly required.

Do not use React.

Do not build a generic dashboard first and defer the 3D. The MVP must include the 3D track and moving cars.

---

# 5. Required Application Experience

The app should look like a polished web product mockup/prototype.

Required visible elements:

## 5.1 Main 3D Race View

A full 3D stylised race track rendered in the browser.

Must include:

- a race circuit/track
- start and finish area
- several cars on the track
- cars positioned by calculated progress
- cars animated smoothly toward their target positions
- checkpoint/sector markers
- pit lane area
- at least one blocked/pit team if CSV contains failed controls
- readable team labels
- modern dark/neon arcade-inspired visual style

The 3D cars can initially be simple stylised meshes. They do not need imported 3D car models in the first version.

Important: do not implement real racing physics. Cars should follow a deterministic spline/path.

Use:

- spline path
- progress value from 0.0 to 1.0
- lane offset
- interpolated position
- tangent-based rotation

Do not use:

- collision simulation
- real vehicle physics
- pathfinding
- random movement as source of truth

## 5.2 Live Leaderboard Panel

Show ranked lanes/teams.

Each row must display:

- rank
- team/lane name
- score percentage
- progress bar
- current checkpoint
- repo count
- status: on-track, pit, blocked, finished

## 5.3 Event Feed Panel

Show derived events such as:

- team entered pit lane
- team is blocked by failed metric
- team completed checkpoint
- team is leading
- team overtook another team, if historical state or simulated previous state exists

For the first version, generate deterministic events from current CSV state if there is no historical feed.

## 5.4 Race Control Summary

Show:

- total repos
- active lanes/teams
- number of metrics
- number of failed controls
- last update time
- CSV load status

## 5.5 Repo/Team Drilldown

At minimum, clicking or selecting a team should show details:

- repos in that lane
- repo score
- failed controls
- current metric group completion
- blockers

This can be a side panel or modal.

---

# 6. Data Model

Design a CSV schema suitable for the prototype.

Start with one denormalised CSV file unless you decide that multiple CSV files are cleaner.

The initial CSV must support fields equivalent to:

lane_id
lane_name
repo
repo_class
criticality
metric_group
metric_name
status
weight
freshness_days
blocker

Allowed status values:

missing
claimed
partial
verified
failed

Suggested score values:

missing = 0.0
claimed = 0.3
partial = 0.6
verified = 1.0
failed = 0.0

Freshness factor:

days <= 7    -> 1.0
days <= 30   -> 0.7
days <= 90   -> 0.3
days > 90    -> 0.0

Metric score:

metric_score =
  status_score
  * metric_weight
  * repo_criticality
  * freshness_factor

Metric max score:

metric_max_score =
  metric_weight
  * repo_criticality

Lane/team score:

lane_score =
  sum(metric_score)
  / sum(metric_max_score)

Lane progress:

progress =
  clamp(lane_score, 0, 1)

Leaderboard:

rank lanes by progress descending

Pit status:

lane is in pit if any metric status is failed or any blocker is present

Finished status:

lane is finished if progress >= 1.0 and no blockers exist

---

# 7. Checkpoint Mapping

Use these default SDLC/DevSecOps race checkpoints:

1. Ownership
2. CI
3. SAST
4. Dependency Hygiene
5. Secrets Scan
6. SBOM
7. Production Readiness

Map metric groups to checkpoints.

Default metric_group values:

ownership
ci
sast
dependency
secrets
sbom
readiness

Infer current checkpoint from progress.

Example mapping:

0.00-0.10 -> Start
0.10-0.25 -> Ownership
0.25-0.40 -> CI
0.40-0.55 -> SAST
0.55-0.68 -> Dependency Hygiene
0.68-0.80 -> Secrets Scan
0.80-0.95 -> SBOM
0.95-1.00 -> Production Readiness / Finish

The implementation should make this mapping configurable in code.

---

# 8. Required Architecture

Use a clean modular architecture.

Suggested structure:

src/
  lib/
    race/
      types.ts
      constants.ts
      csv.ts
      validation.ts
      scoring.ts
      aggregation.ts
      events.ts
      track.ts
      stores.ts
    components/
      RaceScene.svelte
      RaceTrack.svelte
      RaceCar.svelte
      CheckpointMarker.svelte
      PitLane.svelte
      Leaderboard.svelte
      EventFeed.svelte
      RaceControl.svelte
      TeamDetails.svelte
  routes or app entry files
public/
  race.csv

Adjust structure as needed for the generated Svelte/Vite app, but preserve the separation of concerns.

Hard rules:

- CSV parsing must not be inside 3D components.
- Scoring logic must not be inside UI components.
- Three.js/Threlte components must receive already-computed race state.
- Domain types must be explicit.
- Avoid `any`.
- Avoid global mutable state except through controlled Svelte stores.
- Keep renderer independent from data source.
- Keep scoring engine deterministic and unit-testable.
- Keep track/path logic separate from scoring logic.
- Keep CSS readable and maintainable.

---

# 9. 3D Rendering Requirements

Implement a browser-rendered 3D scene.

The track should use a deterministic curve, preferably a CatmullRomCurve3 or equivalent.

Each car must:

- have a lane/team identity
- follow the track spline
- use lane offset so cars do not overlap
- lerp smoothly to target position
- rotate to face track tangent
- display status with color or visual indicator
- show a label above or near the car

Recommended car status visuals:

on-track -> blue/green/purple normal styling
pit -> red styling
blocked -> warning/red pulse
finished -> gold/green highlight

Use simple geometry for MVP:

- boxes/capsules for cars
- plane/tube/mesh for track
- signs/markers for checkpoints

Do not spend time on imported 3D models unless the base prototype is already complete.

---

# 10. UI/UX Requirements

Use a dark arcade-style dashboard aesthetic.

The interface should include:

- large top title: DevSecOps Grand Prix
- subtitle: Live Fleet Progress
- main 3D track area
- right-side leaderboard
- right-side or lower event feed
- left or top race control/status area
- selected team detail panel
- clear loading/error states
- readable typography
- responsive enough for desktop

Prioritise clarity over visual gimmicks.

The app should be impressive enough to pitch to stakeholders, but still technically clean.

---

# 11. Anti-Gaming and Real-World Readiness

Even though the first version uses CSV, include the real-world governance model in the architecture notes.

The system should eventually prevent:

- manually marking tasks complete without evidence
- easy-repo farming
- stale evidence staying green forever
- low-risk repo gaming
- ignoring critical repos
- hiding blocked repos
- regressions not affecting progress

In the code or documentation, make clear that the CSV represents normalised evidence, not raw manual opinions.

---

# 12. Deliverables Required

You must produce the following:

## 12.1 Refined Product/Architecture Brief

First, refine and enhance the idea.

Identify:

- best MVP scope
- assumptions
- risks
- improved data model
- scoring model
- visual model
- implementation plan

Do not ask unnecessary clarifying questions. Make sensible assumptions and proceed.

## 12.2 Working Implementation

Generate the complete working Svelte/Vite/TypeScript application.

Include:

- package.json
- tsconfig/vite config as needed
- source files
- sample public/race.csv
- all required components
- scoring logic
- CSV parser
- 3D scene
- leaderboard
- event feed
- race control
- team details
- basic styling

The project must be runnable.

## 12.3 README

Create a README with:

- what the app does
- how to install
- how to run
- CSV schema
- scoring explanation
- how to add teams/repos/metrics
- architecture overview
- future integration path

## 12.4 Implementation Notes

Include notes for future production hardening:

- replacing CSV with API
- adding WebSocket live updates
- integrating GitHub/GitLab
- integrating CI/CD
- integrating scanners
- storing historical snapshots
- replay mode
- auditability
- evidence provenance

---

# 13. Build Quality Requirements

The implementation must be:

- clean
- modular
- strongly typed
- deterministic
- easy to extend
- easy to visually improve
- suitable as a foundation for production
- free of obvious dead code
- free of mock logic leaking into renderer internals

Avoid:

- overengineering
- hidden magic
- brittle component coupling
- hardcoded values scattered everywhere
- making the 3D scene responsible for business logic
- making CSV parsing responsible for scoring
- making UI state responsible for domain rules

---

# 14. Suggested Sample CSV

Include a realistic sample CSV with at least:

- 4 lanes/teams
- 8-12 repos
- all 7 checkpoint groups
- mixed statuses
- one team in pit lane
- one near finish
- one mid-pack
- one weak/legacy team

Use team names like:

Team Falcon
Team Atlas
Team Kraken
Team Legacy

Use repo names like:

auth-api
billing-worker
portal-ui
deployment-service
identity-gateway
payments-api
legacy-admin
reporting-engine
catalog-service
platform-tools

---

# 15. Expected Race Semantics

Example:

Team Falcon:
- high score
- leading
- near finish
- mostly verified controls

Team Atlas:
- second place
- strong but missing some controls

Team Kraken:
- mid-pack
- some partial controls

Team Legacy:
- low score
- in pit lane due to failed CI or critical CVE

The leaderboard and car positions must reflect the CSV-derived scores.

---

# 16. Implementation Strategy

Work in this order:

1. Create the project skeleton.
2. Define domain types.
3. Define constants/checkpoints.
4. Implement CSV parsing.
5. Implement validation.
6. Implement scoring.
7. Implement lane aggregation.
8. Implement event derivation.
9. Implement Svelte stores.
10. Implement leaderboard and race control UI.
11. Implement 3D track.
12. Implement cars on spline.
13. Implement checkpoint markers.
14. Implement pit lane visual.
15. Implement selected team details.
16. Polish styling.
17. Add README.
18. Review for gaps.

Do not build the visual shell before the domain model works.

Do not build advanced rendering before car movement works.

---

# 17. Acceptance Criteria

The implementation is acceptable only if:

- `npm install` works.
- `npm run dev` starts the app.
- The app loads CSV from `public/race.csv`.
- The leaderboard renders from CSV-derived data.
- The 3D scene renders in browser.
- Cars appear on the track.
- Car positions are based on computed progress.
- Pit lane status is visible for blocked teams.
- At least one team appears in pit lane when CSV has failed/blocker data.
- Checkpoint markers are visible.
- Selecting a team shows useful repo/control detail.
- The scoring formula is documented.
- The data-source boundary is clean.
- The code is TypeScript-safe and modular.

---

# 18. Future Enhancements To Design For

Do not fully implement these unless easy, but design so they are possible:

- live WebSocket updates
- historical replay
- race seasons
- ghost car from previous sprint
- team-specific camera focus
- broadcast camera mode
- repo-level drilldown race
- GitHub/GitLab ingestion
- CI/CD ingestion
- SAST/DAST ingestion
- SBOM validation
- vulnerability feeds
- evidence provenance
- audit logs
- role-based views
- stakeholder wallboard mode

---

# 19. Output Format

Produce your response in this order:

1. Refined concept and architecture summary.
2. Implementation plan.
3. Complete project files.
4. README.
5. Future hardening notes.
6. Final checklist.

When providing code, include clear file paths before each file.

Do not omit files that are required for the project to run.

Do not say “left as an exercise”.

Do not defer the 3D scene.

Do not produce pseudocode where working code is required.

Do not ask for confirmation before building the first version.

Proceed with the strongest reasonable assumptions and build the prototype.