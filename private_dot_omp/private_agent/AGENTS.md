# Global OMP Orchestration

- Use OMP agent orchestration by default when work contains substantial independent slices. Own the decomposition, define shared contracts first, and fan out the independent work in one batch.
- Run independent editing agents with filesystem isolation. Read-only research does not require isolation, and tightly coupled edits that repeatedly touch the same symbols should remain with one owner.
- Keep the main agent moving on the primary task while background agents work. Do not wait when other actionable work remains.
- Give every agent a complete standalone assignment with exact scope, committed inputs, acceptance criteria, and the required verification. Require agents to finish their assigned deliverable rather than return scaffolding or a plan.
- Treat unrelated, confirmed work as a potential side quest instead of expanding the primary diff. Check existing pull requests and branches before spawning it. When repository policy permits autonomous Git operations, let an isolated side-quest agent create its own branch and pull request; never merge it automatically or push directly to a protected branch.
- Review and verify every agent result before reporting completion. Respect stricter repository-specific safety, Git, validation, and ownership rules.
