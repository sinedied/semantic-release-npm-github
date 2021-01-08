# :robot: semantic-release-npm-github

[![NPM version](https://img.shields.io/npm/v/semantic-release-npm-github.svg)](https://www.npmjs.com/package/semantic-release-npm-github)
[![Build Status](https://github.com/sinedied/semantic-release-npm-github/workflows/release/badge.svg)](https://github.com/sinedied/semantic-release-npm-github/actions)
[![XO code style](https://img.shields.io/badge/code_style-XO-5ed9c7.svg)](https://github.com/sindresorhus/xo)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

> Shareable configuration automated package publication to NPM and GitHub using [semantic-release](https://github.com/semantic-release/semantic-release), tailored for OSS projects.

## Release workflow

- [Analyzes](https://github.com/semantic-release/commit-analyzer) commits following the [conventional commits spec](https://www.conventionalcommits.org/)
  * Follows the default [angular preset](https://github.com/semantic-release/commit-analyzer#options)
  * Includes `chore`, `docs`, `refactor` and `style` changes in PATCH releases
- Generates or updates [changelog](https://github.com/semantic-release/changelog)
- Bumps the version in `package.json`
- Publishes package to [NPM](https://npmjs.org)
- Commits the changes made and creates a [git tag](https://github.com/semantic-release/git) with the release version
- Creates a [GitHub release](https://github.com/semantic-release/github) with the package

## Install

1. Install `semantic-release`

  ```sh
  npm install --save-dev semantic-release
  ```

2. Install this package:

  ```sh
  npm install --save-dev semantic-release-npm-github
  ```

3. Add a semantic release config in your `package.json` file:

  ```json
  {
    "extends": "semantic-release-npm-github",
    "branch": "main"
  }
  ```

## Usage

Once everything is installed, you can test your config with a dry run:

```sh
npx semantic-release --dry-run
```

What you'll probably want to do next is configure a [GitHub workflow](https://docs.github.com/actions/quickstart) to run your tests and publish new versions automatically.

Here's a example workflow configuration that runs your tests and publishes a new version for new commits on `main` branch:

```yml
name: release
on:
  push:
    branches:
      - main

jobs:
  test:
    name: Run tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: '>=14'
      - run: |
          npm ci
          npm test
        env:
          CI: true

  release:
    name: Publish release
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v1
        with:
          node-version: '>=14'
      - run: |
          npm ci
          npm build --if-present
        env:
          CI: true
      - run: npx semantic-release
        if: success()
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

In addition, for this workflow to work correctly you have to generate an [NPM authentication token](https://docs.npmjs.com/cli/token) and set it to the [`NPM_TOKEN` secret](https://docs.github.com/actions/reference/encrypted-secrets) in your GitHub repository.

### Note on GitHub protected branches

If you're releasing a [GitHub protected branch](https://docs.github.com/github/administering-a-repository/about-protected-branches) you need to change the git commiter to an owner/admin and allow repo admins to bypass the branch protection (make sure "include administrators" is **disabled** in the branch protection rules.)

If your repo is under an organisation, you can create a bot account and give it admin rights on the repo. If your repo is under a personal account, you have no choice to make the repo owner the commiter for the release.

Either way, you have to create a [GitHub personal access token](https://docs.github.com/github/authenticating-to-github/creating-a-personal-access-token) for the commiter account and give it the "repo" access rights. Then set it to the [`GH_TOKEN` secret](https://docs.github.com/actions/reference/encrypted-secrets) in your GitHub repository.

Finally, make these two changes to your workflow:

```yml
...
    - uses: actions/checkout@v2
      # Add this to commit with a different account than the one
      # used for checkout
      with:
        persist-credentials: false
...
    - run: npx semantic-release
      if: success()
      env:
        # Change the secret used here
        GITHUB_TOKEN: ${{ secrets.GH_TOKEN }}
        NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        # Add this to set the new commiter for the release
        GIT_COMMITTER_NAME: admin-or-owner
        GIT_COMMITTER_EMAIL: associated-email@address.com
```

> Note: GitHub secrets not shared with forks and pull requests, so no one that doesn't have write access to your repo can use of them.
