# PR Submission Process

## 1. Code author creates pull request

- Reference Github issue (if there is one).
- PR should have a small, well-defined scope, complimented with a verbose description.
- Code author to complete the PR submission template.
- Include the following reports:
    - [Foundry gas reports](https://book.getfoundry.sh/forge/gas-reports.html). If updating pre-existing code, please include both before and after reports
    - [Slither](https://github.com/crytic/slither)
    - [SMTChecker](https://docs.soliditylang.org/en/latest/smtchecker.html) or other analysis if conducted
- General note on technical documentation, as much as possible, sensibly include the following:
    - “Swimlane” diagrams
    - Architectural diagrams
    - Wiki documentation
    - READMEs
- Include link to recorded video walkthrough.
- Assign a priority tag to the PR if it needs to be done faster than 3 days.
- Assign reviewers and notify them directly (via Slack).
- Consider inviting persons outside the smart contract team (i.e. front-end) or non devs (business, marketing, etc) when demoing a new feature or an important change.
- Commits should be clearly and descriptively titled.

## Notes on Testing: (for discussion with Richie)

### Unit Testing

Should be isolated to a specific code path/failure mode within a single function in a single contract. For each function:
• Test each failure mode in a dedicated test
• Test each code path in a dedicated test
• Perform before and after assertions of **ALL** affected states

### Fuzz Testing

Test arithmetic calculations and edge cases - for successful code paths that manipulate state in some way or perform calculations.

- Functions that involve arithemetic calculations should be fuzzed, as opposed to using a specific "well-behaved" value when testing.
- Expose any rounding errors, and edge cases due to unexpected values.

### Invariant Testing

Define mathematical (logical) conditions/relationships; these are conditions that should always be true, no matter what.

- If these are ever wrong, it is a critical issue.
- For example, total supply of fyTokens for a vault = sum of all minted by users.

### Economic Testing

Objective is to expose any (mis)understandings/communications between development and business teams.

- Use of scripting to simulate scenarios to ensure the business team's theoretical expectation of product is in-line with developement work.
- Perform multiple full end to end simulations.
- Compare ending state with expected values from business team.

### Integration Testing

- Utilise a Tenderly fork of mainnet.
- Assert all state changes, across all contracts that are affected.
- Consider doing scenario testing, where multiple actions are performed with assertions after each function call.

## 2. Recorded Code Walkthrough  

### Code author will record a video walkthrough with no audience

- This supports an async working environment.  
- Explain the context and significance of change (demo where possible).
- For bug fixes, present a before and after for clear comprehension.
- Highlight areas of risk and complexity.
- Where necessary, tests should be explained - if they assist reviewer in comprehesion, or otherwise contain complexity themselves.
- Upload video to Yield google drive folder: _Google drive link_ (Filename synthax: PR_name_ddmmyy)
- Following this, reviewers may choose to arrange for a follow-up video call if required.

> Discussion with Richie: creating a yield g drive folder for video uploads.

##### Recording Tips

- Breakdown the walkthrough into multiple parts and record them individually - this makes it an easier process for yourself.
- Approach the filming process as if you were doing a video tutorial: use of annotations, callouts, and other call to actions features.
- When combining the various parts together, do include visual seperators and bookmarks for ease of navigation. This is especially useful if its a long video.

**Free screen recorder software:**

- [Free Screen Recorder](https://www.apowersoft.com/free-online-screen-recorder)
- [Xbox Game bar](https://www.theverge.com/2020/4/21/21222533/record-screen-pc-windows-laptop-xbox-game-bar-how-to) (_Windows users_)
- [Camtasia](https://www.techsmith.com/download/camtasia/) (_30-day trial. Just remove and reinstall after_)
- [OBS](https://obsproject.com/)

> From my experience, Camtasia has been wonderfully easy for a new user to jump in and get started. OBS while versatile and free, has an initial steepness to the learning curve.

## 3. Final approval and merging of PR

### The following must be completed by the code author before merging PR:

- Address any comments or changes requested, be sure to mark comments as resolved.
- Notify reviewers if any changes are made to obtain final approval.
- Must have at least 1 approval prior to merge.
- Merge PR and close related Github issue.

```
**For discussion with Richie:**
- only tech lead should be able to merge; if unavailable, provide written permission.
- use of (branch protections)[https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches]
- else developers can use feature branch to unblock any code necessary.
```

## For Reviewers
> Consider using [Github Pull Requests and Issues](https://marketplace.visualstudio.com/items?itemName=GitHub.vscode-pull-request-github) VS code extension

The implementer is only the first line of defense. As a reviewer, confirm that the implementer followed the Yieldcurity principals (test-per-state-transition, test-per-revert, fuzz test, and integration test). Review the documentation and ensure the implementation matches the documented behavior. If it does not, touch base with the implementer and confirm what needs to be updated.

- Reviewers should conduct an independent, thorough review.
- Ideally the review should be completed within 3 days of being notified (faster if priority tagged).
- Refer to the [Yield smart contract code quality standards](/yieldcurity_standard.md) code quality standards.

#### Make note of the kinds of variables that affect the feature:

- User input
- Existing state
- Other protocols
- Time

### General guidelines

- Construct a mental model of what you expect the contracts to look like before checking out the code.
    - Investigate areas that are surprising or deviate from expectations.
- Flesh out the [risk surface](https://github.com/runtimeverification/verified-smart-contracts/wiki/List-of-Security-Vulnerabilities) (i.e. external contract interactions, adversarial actions).
    - Are new attack vectors introduced (if so, have they been mitigated)?
    - Are prior mitigated attack vectors, now unmitigated?
    - Look at areas that can involve **value exchange**.
    - Look at areas that **interface with external contracts**; ensure all assumptions about them are valid.

- Conduct another review from the perspective of an adversary with complete insider knowledge.
- Consider these additional Yield security precautions:
   - Never ever allow users to call arbitrary contracts from our contracts.
   - Either use auth or some kind of whitelist.

- Before reviewing tests, construct a mental model of how you would approach testing:
    - Are realistic scenarios/values used in tests?
    - Are all important edge cases and states being tested?
    - For bug fixes, is there a new test that would have caught the bug?
    - If there are any issues that should hold up merge, mark the PR as Changes Requested.

> Other references: see [Code](/yieldcurity_standard.md#code), [Contract](/yieldcurity_standard.md#contract), and [DeFi](/yieldcurity_standard.md#defi) checklists.

# 4. Post-deployment

## Critical Systems Monitoring

Perform automated checks against deployed contracts

- Use automated monitoring for any “suspicious” behaviour against contracts (Tenderly, OZ Defender)
- Use Pager Duty for critical alerts, web hooks for informational alerts
- Use cron jobs for invariant monitoring
    - Ensure system logics remained unchanged
    - Use atomic “health checker” contracts with view functions to check invariants

## Mainnet Transactions

Safely interact with your deployed contracts

- Simulate mainnet transactions, asserting resulting state on testnet or private fork before executing them on mainnet
- Especially true for administrative operations
