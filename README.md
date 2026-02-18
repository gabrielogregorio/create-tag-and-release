<h1 align="center">Create tag and release</h1>
<p align="center">
  <br>
  <a href="https://github.com/marketplace/actions/create-tag-and-release"><strong>Marketplace »</strong></a>
</p>

This action automates the release and tag creation process, using the github api [https://docs.github.com/pt/rest/releases/releases?apiVersion=2022-11-28#create-a-release](https://docs.github.com/pt/rest/releases/releases?apiVersion=2022-11-28#create-a-release)

## Using
### Prerequisites

You need to configure your repository to allow this action to create the release and the tag, just go to settings, tag and release creation, just go to your settings, actions and general as shown in the screenshot below

![./docs/access-config-actions.png](./docs/access-config-actions.png)

And then just go to the "Workflow permissions" section, check the "Read and write permissions" option and save. Now just re-execute this action

![./docs/select-permissions-workflows.png](./docs/select-permissions-workflows.png)

### Configuring a simpler action

Create a `.yml` file in the `.github/workflows` directory and configure the `gabrielogregorio/create-tag-and-release@v1.0.1` action specifying the version you want to use.

Below is an example of how to use it with some more basic parameters. If you use the `master` branch or another branch, you will need to rename all references to main in the example below

```yml
name: create tag and release on merge pr

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  create-release:
    if: github.event_name == 'pull_request' && github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          ref: main

      - name: create tag and release
        uses: gabrielogregorio/create-tag-and-release@v2.0.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: v2.5.0
          release_name: Release 2.5
          body: |
            A example body


```

### Configuring a more complete action

The example below is much more complete. In the `set config tag and release` step, various information is obtained, such as the version in `package.json`, the title of the pull request and its content, this is used to create releases in a pattern that I like.

```yml
# when a pull request is closed pointing to main
name: create tag and release on merge pr

on:
  pull_request:
    types: [closed]
    branches: [main]

jobs:
  create-release:
    if: github.event_name == 'pull_request' && github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          ref: main

      - name: set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20.x

      - name: set config tag and release
        run: |
          PACKAGE_VERSION=$(node -p "require('./package.json').version")

          PR_TITLE="${{ github.event.pull_request.title }}"
          PR_BODY="${{ github.event.pull_request.body }}"
          BODY_RELEASE="${PR_TITLE} ${PR_BODY}"

          echo "BODY_RELEASE<<EOF" >> $GITHUB_ENV
          echo "$BODY_RELEASE" >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV

          TAG_NAME="v${PACKAGE_VERSION}"
          echo "TAG_NAME=$TAG_NAME" >> $GITHUB_ENV

          RELEASE_NAME="Release v${PACKAGE_VERSION}"
          echo "RELEASE_NAME=$RELEASE_NAME" >> $GITHUB_ENV

      - name: create tag and release
        uses: gabrielogregorio/create-tag-and-release@v2.0.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ env.TAG_NAME }}
          release_name: ${{ env.RELEASE_NAME }}
          body: |
            ${{ env.BODY_RELEASE }}
          make_latest: true
          generate_release_notes: false
          draft: false
          prerelease: false
```
### Understanding Input Settings

The input parameters are inside `with`, example below

```yml
- name: create tag and release
  uses: gabrielogregorio/create-tag-and-release@v1.0.1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    tag_name: ${{ env.TAG_NAME }}             // input
    release_name: ${{ env.RELEASE_NAME }}     // input
    body: ${{ env.BODY_RELEASE }}             // input
    make_latest: true                         // input
    generate_release_notes: false             // input
    draft: false                              // input
    prerelease: false                         // input
    discussion_category_name: 'Announcements' // input
```

And they can be these values

- `tag_name`: (REQUIRED) The name of the tag.
- `release_name`: The name of the release.
- `body`: body of release
- `body_path`: file that will contain the release body. If informed, body will be disregarded
- `draft`: true to create a draft (unpublished) release, false to create a published one. Default: false
- `prerelease`:true to identify the release as a prerelease. false to identify the release as a full release. Padrão: false
- `target_commitish`: Specifies the commitish value that determines where the Git tag is created from. Can be any branch or commit SHA. Unused if the Git tag already exists. Default: the repository's default branch.
- `owner`:The account owner of the repository. The name is not case sensitive. Default Current repository
- `repo`:The name of the repository without the .git extension. The name is not case sensitive. Default Current owner
- `discussion_category_name`: If specified, a discussion of the specified category is created and linked to the release. The value must be a category that already exists in the repository. For more information, see "Managing categories for discussions in your repository."
- `generate_release_notes`:Whether to automatically generate the name and body for this release. If name is specified, the specified name will be used; otherwise, a name will be automatically generated. If body is specified, the body will be pre-pended to the automatically generated notes. Padrão: false
- `make_latest`:Specifies whether this release should be set as the latest release for the repository. Drafts and prereleases cannot be set as latest. Defaults to true for newly published releases. legacy specifies that the latest release should be determined based on the release creation date and higher semantic version. Padrão: true


### Outputs
This action will return the following outputs.

- id
- status
- url
- assets_url
- upload_url
- html_url
- author
- tag_name
- target_commitish
- name
- draft
- prerelease
- assets
- body
- discussion_url
- published_at
- created_at

## Contributing

You can contribute by opening issues, discussions and even pull requests, fixing bugs or adding features that make sense for this project.

To contribute to pull requests, you need to configure node and yarn according to the package.json specifications. Additionally, you need to follow eslint, the commit pattern used in the project, build the application with your changes and open a pull request pointing to the dev branch.
