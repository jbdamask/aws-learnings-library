# AWS Learnings Library

Hard-won lessons from building AWS apps with coding agents — packaged in [llms.txt](https://llmstxt.org/) format so AI agents can pull in just the lessons relevant to the task at hand instead of swallowing a giant monolithic doc.

Each lesson is a small, self-contained Markdown file describing one non-obvious AWS gotcha: the symptom, the root cause, and the fix. An index file (`llms.txt`) links to every lesson so an agent can scan one-line summaries, then fetch only what it needs.

## Layout

```
.
├── llms.txt              # the index: grouped by service, one bullet per lesson
├── lessons/              # one self-contained lesson per file
│   ├── apigw-001.md
│   ├── lambda-001.md
│   └── ...
└── README.md
```

This repo is pure content. The Claude Code skills that read from and contribute to it live in the [`john-claude-skills`](https://github.com/jbdamask/john-claude-skills) marketplace, so they're active across all your projects rather than only when you're working inside this repo — see [Using the library](#using-the-library).

### The index — `llms.txt`

Grouped by AWS service (API Gateway, Lambda, IAM, S3, CloudFront, CloudFormation, Secrets Management, Frontend, EC2, EventBridge, SQS, Spot Interruptions). Each entry is a fully-qualified **raw** URL plus a one-sentence summary, so an agent can decide relevance from the index alone and then fetch the file directly:

```
- [Methods vs Resources are different things](https://raw.githubusercontent.com/jbdamask/aws-learnings-library/main/lessons/apigw-001.md): Resources define URL paths but you also need Methods with Lambda integrations, or you get "Missing Authentication Token".
```

### A lesson — `lessons/<service>-<NNN>.md`

Each lesson carries YAML frontmatter (for machine-readability) followed by the lesson body:

```markdown
---
id: apigw-001
title: API Gateway Methods vs Resources Are Different Things
services: [API Gateway]
summary: Resources define URL paths but you also need Methods with Lambda integrations, or you get "Missing Authentication Token".
---

# API Gateway Methods vs Resources Are Different Things

**Problem:** ...
**Root Cause:** ...
**Solution:** ...
**Key Insight:** ...   # optional
```

Lesson IDs are `<service>-<NNN>`. New contributions add a two-digit seconds suffix (`<service>-<NNN>-<SS>`) to avoid collisions between concurrent contributors — see [Contributing](#contributing).

## Using the library

### From an AI agent (any tool)

Fetch the index and read only the sections matching the AWS services in your current task:

```
https://raw.githubusercontent.com/jbdamask/aws-learnings-library/main/llms.txt
```

Then fetch the individual lesson URLs you need. The point of the llms.txt format is selective retrieval — don't pull every lesson, just the relevant ones.

### From Claude Code (skills)

Two skills in the [`john-claude-skills`](https://github.com/jbdamask/john-claude-skills) marketplace drive this library. Install that marketplace and they're active in every project:

- **`aws-learnings`** (read-only) — triggers when you're creating or modifying AWS infrastructure. It fetches `llms.txt`, identifies the services in play, and applies the relevant lessons proactively, citing them by id.
- **`aws-learnings-add`** (write) — triggers after you've debugged a fresh AWS gotcha worth keeping. It generalizes the lesson, writes the new `lessons/` file, updates `llms.txt`, commits on a branch, and opens a pull request.

## Contributing

Lessons land via pull request so they can be reviewed before joining the library.

The easiest path is the **`aws-learnings-add`** skill, which automates the whole flow. If you'd rather do it by hand:

1. Pick the service prefix and the next sequence number for it, then append the current two-digit seconds: `id = <service>-<NNN>-<SS>` (e.g. `apigw-006-42`). The seconds suffix keeps two simultaneous contributors from grabbing the same id.
2. Create `lessons/<id>.md` with the frontmatter + body shape shown above. Generalize away project-specific names, account IDs, ARNs, and never include secrets.
3. Add a matching bullet under the right `## <Service>` heading in `llms.txt`, using the fully-qualified raw URL for the new file.
4. Commit on a branch (never directly to `main`) and open a PR.

Good lessons lead with the observable **symptom**, explain **why** it happens (not just the fix), and include a minimal, generalized code/CLI snippet.
