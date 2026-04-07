Run the RELEASE stage of the AI Workflow Playbooks pipeline.

Load and execute:
1. `playbooks/release-orchestration/PLAYBOOK.md` — Staged rollout with pre-launch checklist and go/no-go gates

Follow the pre-launch checklist:
- All tests pass
- Security scan is clean
- Documentation is updated
- Monitoring and alerting are configured
- Rollback procedure is verified

Determine go/no-go. If go, execute the release plan with appropriate staging (canary → percentage rollout → full release). Verify post-launch success criteria.
