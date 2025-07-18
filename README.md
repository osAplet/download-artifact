# `@actions/download-artifact`

Download [Actions Artifacts](https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts) from your Workflow Runs. Internally powered by the [@actions/artifact](https://github.com/actions/toolkit/tree/main/packages/artifact) package.

See also [upload-artifact](https://github.com/actions/upload-artifact).

- [`@actions/download-artifact`](#actionsdownload-artifact)
  - [v4 - What's new](#v4---whats-new)
    - [Improvements](#improvements)
    - [Breaking Changes](#breaking-changes)
  - [Usage](#usage)
    - [Inputs](#inputs)
    - [Outputs](#outputs)
  - [Examples](#examples)
    - [Download Single Artifact](#download-single-artifact)
    - [Download Artifacts by ID](#download-artifacts-by-id)
    - [Download All Artifacts](#download-all-artifacts)
    - [Download multiple (filtered) Artifacts to the same directory](#download-multiple-filtered-artifacts-to-the-same-directory)
    - [Download Artifacts from other Workflow Runs or Repositories](#download-artifacts-from-other-workflow-runs-or-repositories)
  - [Limitations](#limitations)
    - [Permission Loss](#permission-loss)


## v4 - What's new

> [!IMPORTANT]
> download-artifact@v4+ is not currently supported on GHES yet. If you are on GHES, you must use [v3](https://github.com/actions/download-artifact/releases/tag/v3).

The release of upload-artifact@v4 and download-artifact@v4 are major changes to the backend architecture of Artifacts. They have numerous performance and behavioral improvements.

For more information, see the [`@actions/artifact`](https://github.com/actions/toolkit/tree/main/packages/artifact) documentation.

### Improvements

1. Downloads are significantly faster, upwards of 90% improvement in worst case scenarios.
2. Artifacts can be downloaded from other workflow runs and repositories when supplied with a PAT.

### Breaking Changes

1. On self hosted runners, additional [firewall rules](https://github.com/actions/toolkit/tree/main/packages/artifact#breaking-changes) may be required.
2. Downloading artifacts that were created from `action/upload-artifact@v3` and below are not supported.

For assistance with breaking changes, see [MIGRATION.md](docs/MIGRATION.md).

## Note

Thank you for your interest in this GitHub repo, however, right now we are not taking contributions. 

We continue to focus our resources on strategic areas that help our customers be successful while making developers' lives easier. While GitHub Actions remains a key part of this vision, we are allocating resources towards other areas of Actions and are not taking contributions to this repository at this time. The GitHub public roadmap is the best place to follow along for any updates on features we’re working on and what stage they’re in.

We are taking the following steps to better direct requests related to GitHub Actions, including:

1. We will be directing questions and support requests to our [Community Discussions area](https://github.com/orgs/community/discussions/categories/actions)

2. High Priority bugs can be reported through Community Discussions or you can report these to our support team https://support.github.com/contact/bug-report.

3. Security Issues should be handled as per our [security.md](SECURITY.md).

We will still provide security updates for this project and fix major breaking changes during this time.

You are welcome to still raise bugs in this repo.

## Usage

### Inputs

```yaml
- uses: actions/download-artifact@v4
  with:
    # Name of the artifact to download.
    # If unspecified, all artifacts for the run are downloaded.
    # Optional.
    name:

    # IDs of the artifacts to download, comma-separated.
    # Either inputs `artifact-ids` or `name` can be used, but not both.
    # Optional.
    artifact-ids:

    # Destination path. Supports basic tilde expansion.
    # Optional. Default is $GITHUB_WORKSPACE
    path:

    # A glob pattern to the artifacts that should be downloaded.
    # Ignored if name is specified.
    # Optional.
    pattern:

    # When multiple artifacts are matched, this changes the behavior of the destination directories.
    # If true, the downloaded artifacts will be in the same directory specified by path.
    # If false, the downloaded artifacts will be extracted into individual named directories within the specified path.
    # Optional. Default is 'false'
    merge-multiple:

    # The GitHub token used to authenticate with the GitHub API.
    # This is required when downloading artifacts from a different repository or from a different workflow run.
    # Optional. If unspecified, the action will download artifacts from the current repo and the current workflow run.
    github-token:

    # The repository owner and the repository name joined together by "/".
    # If github-token is specified, this is the repository that artifacts will be downloaded from.
    # Optional. Default is ${{ github.repository }}
    repository:

    # The id of the workflow run where the desired download artifact was uploaded from.
    # If github-token is specified, this is the run that artifacts will be downloaded from.
    # Optional. Default is ${{ github.run_id }}
    run-id:
```

### Outputs

| Name | Description | Example |
| - | - | - |
| `download-path` | Absolute path where the artifact(s) were downloaded | `/tmp/my/download/path` |

## Examples

### Download Single Artifact

Download to current working directory (`$GITHUB_WORKSPACE`):

```yaml
steps:
- uses: actions/download-artifact@v4
  with:
    name: my-artifact
- name: Display structure of downloaded files
  run: ls -R
```

Download to a specific directory (also supports `~` expansion):

```yaml
steps:
- uses: actions/download-artifact@v4
  with:
    name: my-artifact
    path: your/destination/dir
- name: Display structure of downloaded files
  run: ls -R your/destination/dir
```

### Download Artifacts by ID

The `artifact-ids` input allows downloading artifacts using their unique ID rather than name. This is particularly useful when working with immutable artifacts from `actions/upload-artifact@v4` which assigns a unique ID to each artifact.

```yaml
steps:
- uses: actions/download-artifact@v4
  with:
    artifact-ids: 12345
- name: Display structure of downloaded files
  run: ls -R
```

Multiple artifacts can be downloaded by providing a comma-separated list of IDs:

```yaml
steps:
- uses: actions/download-artifact@v4
  with:
    artifact-ids: 12345,67890
    path: path/to/artifacts
- name: Display structure of downloaded files
  run: ls -R path/to/artifacts
```

This will download multiple artifacts to separate directories (similar to downloading multiple artifacts by name).

### Download All Artifacts

If the `name` input parameter is not provided, all artifacts will be downloaded. To differentiate between downloaded artifacts, by default a directory denoted by the artifacts name will be created for each individual artifact. This behavior can be changed with the `merge-multiple` input parameter.

Example, if there are two artifacts `Artifact-A` and `Artifact-B`, and the directory is `etc/usr/artifacts/`, the directory structure will look like this:

```
etc/usr/artifacts/
    Artifact-A/
        ... contents of Artifact-A
    Artifact-B/
        ... contents of Artifact-B
```

Download all artifacts to the current working directory:

```yaml
steps:
- uses: actions/download-artifact@v4
- name: Display structure of downloaded files
  run: ls -R
```

Download all artifacts to a specific directory:

```yaml
steps:
- uses: actions/download-artifact@v4
  with:
    path: path/to/artifacts
- name: Display structure of downloaded files
  run: ls -R path/to/artifacts
```

To download them to the _same_ directory:

```yaml
steps:
- uses: actions/download-artifact@v4
  with:
    path: path/to/artifacts
    merge-multiple: true
- name: Display structure of downloaded files
  run: ls -R path/to/artifacts
```

Which will result in:

```
path/to/artifacts/
    ... contents of Artifact-A
    ... contents of Artifact-B
```

### Download multiple (filtered) Artifacts to the same directory

In multiple arch/os scenarios, you may have Artifacts built in different jobs. To download all Artifacts to the same directory (or matching a glob pattern), you can use the `pattern` and `merge-multiple` inputs.

```yaml
jobs:
  upload:
    strategy:
      matrix:
        runs-on: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.runs-on }}
    steps:
    - name: Create a File
      run: echo "hello from ${{ matrix.runs-on }}" > file-${{ matrix.runs-on }}.txt
    - name: Upload Artifact
      uses: actions/upload-artifact@v4
      with:
        name: my-artifact-${{ matrix.runs-on }}
        path: file-${{ matrix.runs-on }}.txt
  download:
    needs: upload
    runs-on: ubuntu-latest
    steps:
    - name: Download All Artifacts
      uses: actions/download-artifact@v4
      with:
        path: my-artifact
        pattern: my-artifact-*
        merge-multiple: true
    - run: ls -R my-artifact
```

This results in a directory like so:

```
my-artifact/
  file-macos-latest.txt
  file-ubuntu-latest.txt
  file-windows-latest.txt
```

### Download Artifacts from other Workflow Runs or Repositories

It may be useful to download Artifacts from other workflow runs, or even other repositories. By default, the permissions are scoped so they can only download Artifacts within the current workflow run. To elevate permissions for this scenario, you can specify a `github-token` along with other repository and run identifiers:

```yaml
steps:
- uses: actions/download-artifact@v4
  with:
    name: my-other-artifact
    github-token: ${{ secrets.GH_PAT }} # token with actions:read permissions on target repo
    repository: actions/toolkit
    run-id: 1234
```

## Limitations

### Permission Loss

File permissions are not maintained during artifact upload. All directories will have `755` and all files will have `644`. For example, if you make a file executable using `chmod` and then upload that file, post-download the file is no longer guaranteed to be set as an executable.

If you must preserve permissions, you can `tar` all of your files together before artifact upload. Post download, the `tar` file will maintain file permissions and case sensitivity.

```yaml
- name: 'Tar files'
  run: tar -cvf my_files.tar /path/to/my/directory

- name: 'Upload Artifact'
  uses: actions/upload-artifact@v4
  with:
    name: my-artifact
    path: my_files.tar
```
