# action-tests
Test repo to mess around with actions / environments / deployments

This uses 2 workflows
- **Pull Request**: a simple CI workflow executing build and test on the
  PR merge candidate.
- **Build and Release**: the full deployment pipeline that builds, tests, and
  create artifacts. It then deploys to _dev_, _qa_ and finally to all
  _production_ environments. From _qa_ on, each environment is configured with
  approvals required to progress.
