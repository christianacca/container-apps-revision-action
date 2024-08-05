# Contriution guide

## Dev workflow

1. Create a feature branch either from master or from a release branch
2. Make changes and commit to feature branch
3. Commit to feature branch, then push feature branch
4. Create a pull request (PR) from feature branch to master
5. Wait for PR to be approved and then merge
    * **tip**: typically you will perform a squish merge to keep the commit history clean
6. Delete feature branch

## Deploying a new release of this action

1. create full semantic-version tag for current branch. EG:
   ```bash
   # Replace v1.0.2 with the new semantic version (see https://semver.org/)
   git tag v1.0.2
   git push origin v1.0.2
   ````
2. move the current major version tag (eg v1) to reference the new tag version
   ```bash
   # As required replace v1 with the current major version tagged in the repo
   git tag -d v1
   git push --delete origin v1
   git tag v1 $(git rev-parse HEAD)
   git push origin v1
   ````
3. In github
   1. publish the draft (eg v1) release, ensuring that it is NOT set as latest release
   2. create the release (eg v1.0.2) from the new tag created above, ensuring it IS set as latest release