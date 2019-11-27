# POC for testing automated release

https://bagonboard.atlassian.net/wiki/spaces/PRD/pages/1344045061/Automate+releases+versioning+WIP

## Commits

### Conventional commits

Following "conventional commits" convention: Conventional. https://www.conventionalcommits.org/en/v1.0.0-beta.2/

### Commitlint

Enforced commits by usage of commitlint hook with husky. https://www.npmjs.com/package/@commitlint/cli

```
"hooks": {
  "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
}
```
CommitLint config: @commitlint/config-conventional (see .commitlint.config.js)

### Commitizen

Use Commitizen for creating best commits through script. https://www.npmjs.com/package/commitizen

```
"scripts": {
  "commit": "git-cz"
},
```

```
> npm run commit
```

## Automated Releases

### Semantic Release

Using Semantic Release: https://semantic-release.gitbook.io/semantic-release/

And similar configuration to: https://github.com/jedmao/semantic-release-npm-github-config (see .releaserc)

Uses GITHUB_TOKEN environment variable with a personal access token. https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line

Call semantic releases with npm script

```
"scripts": {
  // ci version:
  "release": "semantic-release"
  ...
  // local version 
  // "env-cmd -f .env" to set GITHUB_TOKEN from environment variable
  // "--no-ci" option to allow create releases outside ci
  "release": "env-cmd -f '.env' semantic-release --no-ci"
},
```

Creating release on CI (JenkinsFile)
```
> npm run release
```

### Semantic Release plugins

Plugins Description:

- [@semantic-release/commit-analyzer](https://github.com/semantic-release/commit-analyzer): Determine the type of release by analyzing commits with conventional-changelog.

- [@semantic-release/release-notes-generator](https://github.com/semantic-release/release-notes-generator):  Generate release notes for the commits added since the last release with conventional-changelog.

- [@semantic-release/changelog](https://github.com/semantic-release/changelog): Create or update a changelog file in the local project directory with the changelog content.

- [@semantic-release/github](https://github.com/semantic-release/github): Publish a GitHub release, optionally uploading file assets.

- [@semantic-release/git](https://github.com/semantic-release/git): Create a release commit, including configurable file assets.


## Additional Commit

Semantic release creates an additional commit with the changelog (and the [skip ci] text in the commit message), in order to avoid this commit to launch another deployment, we have added following configuration in Jenkins pipeline:

Set environment variable with "true" value if commit contains [skip ci] text:

```
environment {
  SKIP_PIPELINE = """${sh(
      returnStatus: true,
      script: 'git log -1 --pretty=%B | fgrep -ie "[skip ci]" -e "[ci skip]"',
  ) == 0}"""
}
```

Evaluate condition on stages to be skipped:

```
when {
  expression {
    !env.SKIP_PIPELINE.toBoolean()
  }
}
```

[![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)