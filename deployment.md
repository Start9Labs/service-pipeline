## Guidelines for testing and deployment

### Alpha

For alpha deployments: 

- Coordinate with a specific tester, assign them specific tasks to test within the service UI, and encourage baseline service OS testing
- Versions can be overwritten on alpha
- s9pk can be built from a branch that is not merged into master

### Beta

For beta deployments:

- Treat the s9pk as a release candidate (RC), an asset that can be promoted directly to prod without rebuilding
- Release notes should be complete
- All PRs should be merged to master
- Create a pre-release release tag in GitHub and attach the beta RC s9pk to it
- Increment version (and merge this change to master) whenever a bugfix or adjustment is made
- IN PROGRESS: building GitHub actions for all service wrapper repos such that pushing to master will produce an s9pk that is the RC asset

### Prod

For production deployments:

- Ensure the final s9pk is deployed to beta at the correct version
- Convert the pre-release GitHub release tag to the latest release tag
- Use the s9pk RC asset on this tag to deploy to production