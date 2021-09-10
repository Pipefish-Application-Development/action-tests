# action-tests
Test repo to mess around with actions / environments / deployments

This uses 2 workflows
- **Pull Request**: a simple CI workflow executing build and test on the
  PR merge candidate.
- **Build and Release**: the full deployment pipeline that builds, tests, and
  create artifacts. It then deploys to _dev_, _qa_ and finally to all
  _production_ environments. From _qa_ on, each environment is configured with
  approvals required to progress.



# NOTES

- To build a zip file as an artifact for an Azure Function, you need to run zip
  in the root folder of the function and pass in the exclusions file
  ```
  zip --recurse-paths some/temp/folder/function.zip . --exclude @.funcignore
  ```
  and then you can publish the artifact using
  [actions/upload-artifact](https://github.com/actions/upload-artifact)
- Once published, the artifact is available using the
  [actions/download-artifact](https://github.com/actions/download-artifact) action in other jobs in the _same_ workflow.
- Azure cli is installed in the runner by default, but you may need to install
  the function app tools before you can build/publish from the runner.
- Keep actual work steps in actions, like the build and deployment steps so
  that the workflow file is about coordination, not details.

## Environments

Will need to enable GitHub environments in the subscription for private repos
(this is public so we could experiment first).

Then configure each deployment (from dev, test, qa to production instances) in
the _Settings/Environments_ tab in the repo. You will need to
- Configure Protection rules to ensure approvals before deployments. Note that
  only 1 of the configured users needs to approve for it to progress so make it
  the person who is most responsible for that customer/environment.
- Configure environment secrets to allow access to the infrastructure and other
  settings that need to be propagated into the environment (tenantid etc.)
- Global secrets can also be used if for instance only a single
  Service Principal is used to access all deployments, it can be at the repo
  level, and specific settings like tenantid can be at the environment level.
- For qa/production environments at least, ensure that the deployment branch
  restriction is in place for only the `main` branch to prevent deployment
  of non-released code.

## Developer Workflow

- Enable branch protection on the `main` branch
  - Require Pull Request Reviews
  - Require Status Checks (builds and external auditing tools/integrations) to
    complete before PR can be merged
  - Strongly suggest requiring linear history (no merge commits)
- Developers work in short lived story/feature branch and PR back into `main`
- Merge to `main` after PR accepted will then trigger build/release pipeline to
  generate build artifacts and publish to dev environment for further
  integration testing.
- Test manager can then approve dev to move to test, and then to QA and finally
  customer liaison can approve for each production instance

This is a pretty simple workflow, and can be adapted and refined as team
experience grows.

It may be tempting to add a `release/vx.y` style branch before `main` and have
a release candidate deployment (or reuse dev) where these changes are
completed before triggering the main release workflow, taking the new code
to production (more elements from the GitFlow style process).

However this ends up slowing down the CD process - the released changes should
ideally be small and only take a day or two to be implemented. Use feature
flags to disable new functionality until ready to release publicly. This prevents
integration hell after long running separate feature development branches.

Ensuring that the small changes are constantly integrated into the main line
(and disabled in production until complete) means that code is always up to
date and it minimised merge hell.


