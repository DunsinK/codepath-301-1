# Contribution 1: WinGet Service — Add ReleaseDate

**Contribution Number:** 1  
**Student:** Dunsin Komolafe  
**Issue:** [#11285 — WinGet Service: add ReleaseDate](https://github.com/badges/shields/issues/11285)  
**Status:** Phase 4 — In Progress (waiting for maintainter response) 

---

## Why I Chose This Issue

I chose this issue because it looked well-scoped, self-contained feature addition to an existing service. It was a good fit for a first open source contribution. The WinGet badge already works, so I'm not starting from scratch; instead I need to understand an established pattern and extend it responsibly. That balance of "enough existing structure to learn from, but enough novelty to be a real contribution" felt right. I also wanted hands-on experience with a real-world codebase that uses GraphQL, Joi schema validation, and a test-first culture. This issue requires me to make two GitHub GraphQL calls and parse YAML, which pushes me beyond simple REST badge additions and gives me a more complete picture of how shields.io services are architected.

---

## Understanding the Issue

### Problem Description

The WinGet badge (`shields.io/winget/v/{name}`) currently only exposes the latest version number of a package. WinGet manifests contain a `ReleaseDate` field, but there is no badge endpoint that surfaces it. Users who want to display when a package was last released have no option.

### Expected Behavior

A new badge endpoint — `shields.io/winget/release-date/{name}` — should exist and display the release date of the latest version of a WinGet package, sourced from the manifest YAML in the `microsoft/winget-pkgs` repository.

### Current Behavior

No `release-date` endpoint exists for WinGet. Only `winget/v/{name}` (version) is available.

### Affected Components

- `services/winget/winget-version.service.js` — existing version badge (reference for the pattern)
- `services/winget/` directory — new files will be added here
- GitHub GraphQL API — the data source (the `microsoft/winget-pkgs` repo)

---

## Reproduction Process

### Environment Setup

Forked and cloned the `badges/shields` repository locally. The project requires Node.js and uses `npm` for dependency management. No special environment secrets are needed to browse the WinGet service code; a GitHub token would be needed to run live service tests against the GraphQL API.

### Steps to Reproduce

1. Visit `https://img.shields.io/winget/v/Microsoft.WSL` — the version badge works correctly
2. Visit `https://img.shields.io/winget/release-date/Microsoft.WSL` — no such endpoint exists
3. Observed result: 404 / unknown badge
4. reproduce locally with a github token and npm run badge

### Reproduction Evidence

- **Commit showing reproduction:** [To be added]
- **Screenshots/logs:** N/A — the endpoint simply does not exist
- **My findings:** After reading `winget-version.service.js`, I confirmed the service uses GitHub GraphQL to list version directories in `microsoft/winget-pkgs` but never reads the YAML manifest file contents where `ReleaseDate` lives. A second GraphQL call fetching blob content is required.

---

## Solution Approach

### Analysis

The root cause is simply that no one has built this badge yet. The `winget-version.service.js` service lists directory names via GraphQL but never fetches file contents, so `ReleaseDate` — which lives inside the manifest YAML — is not accessible with the current query. Adding it requires a second GraphQL call that reads the blob content of the `.yaml` file for the latest version.

### Proposed Solution

Create a new `winget-release-date.service.js` that:
1. Reuses the same version-finding logic as `winget-version.service.js` (first GraphQL call)
2. Makes a second GraphQL call to fetch the blob text of the manifest YAML for the latest version
3. Parses `ReleaseDate` from the YAML text
4. Returns the result via `renderDateBadge`

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The WinGet badge needs a new endpoint that displays the release date of the latest version of a package. The date lives in a YAML file inside the `microsoft/winget-pkgs` GitHub repo and is not currently fetched.

**Match:** The multi-badge pattern used by `services/vaadin-directory/` (shared base + separate service files per field) and `services/npm/` is the right model. The `renderDateBadge` helper from `services/date.js` handles date badge formatting. The GitHub GraphQL blob content query (using `... on Blob { text }`) is the mechanism for reading file contents.

**Plan:**
1. Create `services/winget/winget-release-date.service.js` extending `GithubAuthV4Service`
2. In `fetch()` call 1: query directory listing to find the latest version (same logic as `winget-version.service.js`)
3. In `fetch()` call 2: query `HEAD:manifests/{x}/{name}/{version}/{name}.yaml` as a blob to get file text
4. Parse `ReleaseDate: YYYY-MM-DD` from the YAML text (simple regex or `js-yaml` if already a dependency)
5. Return `renderDateBadge({ date })` with `label: 'release date'`
6. Create `services/winget/winget-release-date.tester.js` with live and mocked tests

**Implement:** [Link to branch/commits — to be added as work progresses]

**Review:**
- [ ] Follows kebab-case file naming
- [ ] Class name is PascalCase (`WingetReleaseDate`)
- [ ] `static category` set to `'other'`
- [ ] `static openApi` block included
- [ ] Joi schema validates only the fields used
- [ ] Tests cover live call, mocked happy path, and error cases
- [ ] PR title includes `[WinGet]`

**Evaluate:** Run `npm run test:services -- --only=winget` and confirm both the existing version tests and new release-date tests pass. Manually verify the badge renders correctly at the local dev server.

---

## Testing Strategy

### Unit Tests

- [ ] Valid date is parsed correctly from YAML text
- [ ] Missing `ReleaseDate` field in YAML throws a handled error
- [ ] Package not found (null entries) throws `InvalidParameter`

### Integration Tests

- [ ] Live test: `Microsoft.WSL` returns a badge with label `release date` and a valid date message
- [ ] Mocked test: known YAML fixture returns the exact expected date string
- [ ] Mocked test: package with no versions returns `no versions found`

### Manual Testing

[To be completed — will verify badge renders at local dev server with `npm start`]

---

## Implementation Notes

### Week 1 Progress

Explored the codebase to understand the existing WinGet service before writing any code. Key findings:
- The current service uses GitHub GraphQL (not a REST API) because WinGet packages are YAML files in a GitHub repo
- It only reads directory names/file names — never file contents — so `ReleaseDate` is not accessible with the existing query
- A second GraphQL call using `... on Blob { text }` is needed to read the manifest YAML
- The multi-badge pattern (separate `.service.js` files per field) is the established convention in this codebase
- `renderDateBadge` from `services/date.js` is the correct renderer to use

### Code Changes

- **Files modified:** None yet (Phase I — understanding only)
- **Key commits:** [To be added]
- **Approach decisions:** Decided against creating a shared base class since the two services (version and release-date) require different fetching logic; a standalone new service file is cleaner

---

## Pull Request

**PR Link:** (https://github.com/badges/shields/pull/11919)

**PR Description:** [WinGet] Add release date badge`]

**Maintainer Feedback:**
- "These tests are a little hard to follow. Could we use simpler fixtures, with just the required number of entries needed to exercise the logic?"

**Status:** Still in progress

---

## Learnings & Reflections

### Technical Skills Gained

[To be completed at end of contribution]

### Challenges Overcome

[To be completed at end of contribution]

### What I'd Do Differently Next Time

[To be completed at end of contribution]

---

## Resources Used

- [shields.io CONTRIBUTING.md](https://github.com/badges/shields/blob/master/CONTRIBUTING.md)
- [Issue #11285 — WinGet Service: add ReleaseDate](https://github.com/badges/shields/issues/11285)
- [GitHub GraphQL API docs — querying blob content](https://docs.github.com/en/graphql/reference/objects#blob)
- [shields.io service-tests.md](https://github.com/badges/shields/blob/master/doc/service-tests.md)
- [Vaadin Directory release-date service — pattern reference](https://github.com/badges/shields/blob/master/services/vaadin-directory/vaadin-directory-release-date.service.js)

# Contribution 2: LLama.cpp - support for orpheus

**Contribution Number:** 2  
**Student:** Dunsin Komolafe  
**Issue:** Llama.cpp
**Status:** Phase 2 — In Progress

---

## Why I Chose This Issue

I chose this issue because it looked well-scoped, with just the addition for orpheus support. It was a good fit for a first open source contribution. I also wanted hands-on experience with a real-world codebase that uses C++ and logic for running AI models

---

