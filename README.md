# actions-city-modelling-lab

Reusable GitHub Action workflows for use across City Modelling Lab repositories

## Example usage

You can find more detail on reusable workflows in the [GitHub documentation](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

If you want to add a CI run for your project `my-great-project` whenever a push is made to the repository:

``` yaml
name: CI

on:
  push:
  branches:
  - "**"

jobs:
  test:
    uses: arup-group/actions-city-modelling-lab/.github/workflows/python-install-lint-test.yml@main
    with:
      os: ubuntu-latest
      py3version: "11"
      notebook_kernel: "my-great-project"
      lint: true
```

To do the same across a matrix of operating systems and python versions, maybe when a pull request is opened / ready for review:

``` yaml
name: CI

on:
  pull_request:
    branches:
    - main
    types:
    - opened
    - ready_for_review
    - review_requested

jobs:
  test:
    strategy:
      matrix:
        os: [windows-latest, ubuntu-latest, macos-latest]
        py3version: ["9", "10", "11"]
    uses: arup-group/actions-city-modelling-lab/.github/workflows/python-install-lint-test.yml@main
    with:
      os: ${{ matrix.os }}
      py3version: ${{ matrix.py3version }}
      notebook_kernel: "my-great-project"
      # ignore coverage and linting, those come in the deep-dive below.
      lint: false
      pytest_args: '--no-cov'
      upload_to_codecov: false

  test-deep-dive:  # Run again on ubuntu-latest py3.11 with linting and coverage reporting.
    uses: arup-group/actions-city-modelling-lab/.github/workflows/python-install-lint-test.yml@main
    with:
      os: ubuntu-latest
      py3version: "11"
      notebook_kernel: "my-great-project"
      lint: true
      pytest_args: 'tests/ mkdocs_plugins/'  # ignore example notebooks.
      upload_to_codecov: true
```

You can also chain reusable workflows:

``` yaml
name: CI

on:
  push:
    branches:
    - "**"

jobs:
  test:
    uses: arup-group/actions-city-modelling-lab/.github/workflows/python-install-lint-test.yml@main
    with:
      os: ubuntu-latest
      py3version: "11"
      notebook_kernel: "my-great-project"
      lint: true
  aws-upload:
    needs: test
    if: needs.test.result == 'success'
    uses: arup-group/actions-city-modelling-lab/.github/workflows/aws-upload.yml@main
    secrets: inherit
  slack-notify-ci:
    needs: test
    uses: arup-group/actions-city-modelling-lab/.github/workflows/slack-notify.yml@main
    secrets: inherit
    with:
      result: needs.test.result
      channel: my-great-project-feed
      message: "Commit CI action"
  slack-notify-aws:
    needs: aws-upload
    uses: arup-group/actions-city-modelling-lab/.github/workflows/slack-notify.yml@main
    secrets: inherit
    with:
      result: needs.aws-upload.result
      channel: my-great-project-feed
      message: "AWS upload action"
```

> [!NOTE]
> You can _only_ use `secrets: inherit` if you are hosting your repository in the `arup-group` organisation.
> If you have the repo under your own username, you will need to explicitly pass the necessary secrets, e.g.:
>
> ``` yaml
> secrets:
>   SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
> ```

## Available workflows

### Upload package to AWS

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/aws-upload.yml`

_description_: Upload the current state of the repository as a zipped file to an AWS S3 bucket.
This may then be picked up by AWS CodeBuild to e.g. build a Docker image using the Dockerfile in the zip.

_Inputs_: None

_Required secrets_: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_S3_CODE_BUCKET`

### Build a conda package

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/conda-build.yml`

_description_: Build a conda package and store it as an artefact in your GitHub repository.
This could be used in a release pull request, ready to upload the build to Anaconda in a tagged release of your package.

_Inputs_:

- package_name: Name of your package, as defined in your `pyproject.toml` or `setup.py` file (if your repo is a Python project).
- version (optional, default=the version tag linked to the commit): version of the package you want to build.
- recipe_dir (optional, default="conda.recipe"): Directory in which to find the recipe (i.e. configuration) for building the package.
- environment (optional, default="pre-release"): GitHub environment in which secrets are stored.
Environments help to ensure that only certain operations are available to different user types.
E.g., releasing packages can be given an extra layer of security whereby a maintainer has to approve an action before it can run.
- destination (optional, default="anaconda"): One of "anaconda" or "internal", to specify what the ultimate destination of the package will be.
If `internal`, the package will be uploaded to <https://packages.arup.com/conda>.
If `anaconda`, the package will be uploaded to <https://anaconda.org/[CHANNEL-NAME]/> where `[CHANNEL-NAME]` is linked to the `ANACONDA_TOKEN` secret.

_Required secrets_: `ANACONDA_TOKEN` (required to verify that later upload will not fail) stored in a GitHub actions environment of the same name as `environment`.
If `destination=internal`, this secret must still be defined, but can be a placeholder string (e.g. "NA").

> [!NOTE]
> To use this action with `destination=internal`, you must request access to the `packages` self-hosted runner for your repository via an Arup service-now request.

### Upload a conda package

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/conda-upload.yml`

_description_: Upload a built conda package stored as an artefact in your GitHub repository (see [](#build-a-conda-package)).
This could be used when you publish a new release of your package on GitHub.

_Inputs_:

- package_name: Name of your package, as defined in your `pyproject.toml` or `setup.py` file (if your repo is a Python project).
- version (optional, default=the version tag linked to the commit): version of the package you want to upload.
- build_workflow (optional, default=""): If you have built your package using a job in a _different_ workflow file you will need to give its name here (e.g. `pre-release.yml`).
- environment (optional, default="pre-release"): GitHub environment in which secrets are stored.
Environments help to ensure that only certain operations are available to different user types.
E.g., releasing packages can be given an extra layer of security whereby a maintainer has to approve an action before it can run.
- destination (optional, default="anaconda"): One of "anaconda" or "internal", to specify what the ultimate destination of the package will be.
If `internal`, the package will be uploaded to <https://packages.arup.com/conda>.
If `anaconda`, the package will be uploaded to <https://anaconda.org/[CHANNEL-NAME]/> where `[CHANNEL-NAME]` is linked to the `ANACONDA_TOKEN` secret.

_Required secrets_: `ANACONDA_TOKEN` stored in a GitHub actions environment of the same name as `environment`.
If `destination=internal`, this secret must still be defined, but can be a placeholder string (e.g. "NA").

> [!NOTE]
> To use this action with `destination=internal`, you must request access to the `packages` self-hosted runner for your repository via an Arup service-now request.

### Build a pip package for upload to PyPI or to <https://packages.arup.com>

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/pip-build.yml`

_description_: Build a pip package and store it as an artefact in your GitHub repository.
This could be used in a release pull request, ready to upload the build to PyPI in a tagged release of your package.
The built package will be uploaded to Test-PyPI to allow you to test installing it from PyPI without having actually released the package yet.

To test installing from Test-PyPI: `pip install -i https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple [package-name]==[package-version]`

_Inputs_:

- package_name: Name of your package, as defined in your `pyproject.toml` or `setup.py` file (if your repo is a Python project).
- version (optional, default=the version tag linked to the commit): version of the package you want to build.
- environment (optional, default="pre-release"): GitHub environment in which secrets are stored.
Environments help to ensure that only certain operations are available to different user types.
E.g., releasing packages can be given an extra layer of security whereby a maintainer has to approve an action before it can run.
- pip_args (optional, default="--no-deps"). Any arguments to pass to pip when running test installations.
Many of our packages have non-python dependencies, so it is useful to use `--no-deps` in the installation.
However, if you know that your library has purely python dependencies then the pip build process is made more robust by removing this argument (i.e. `pip_args: ""`)
- destination (optional, default="pypi"): One of "pypi" or "internal", to specify what the ultimate destination of the package will be.
If `internal`, the package will be uploaded to <https://packages.arup.com>.
If `pypi`, the package will be uploaded to <https://test.pypi.org/> for testing and <https://pypi.org/> for final upload.

_Required secrets_: `TEST_PYPI_API_TOKEN` stored in a GitHub actions environment of the same name as `environment`.
If `destination=internal`, this secret must still be defined, but can be a placeholder string (e.g. "NA").

> [!NOTE]
> To use this action with `destination=internal`, you must request access to the `packages` self-hosted runner for your repository via an Arup service-now request.

### Upload a pip package to PyPI or to <https://packages.arup.com>

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/pip-upload.yml`

_description_: Upload a built pip package stored as an artefact in your GitHub repository (see [](#build-a-pip-package-for-upload-to-pypi)).
This could be used when you publish a new release of your package on GitHub.

_Inputs_:

- package_name: Name of your package, as defined in your `pyproject.toml` or `setup.py` file (if your repo is a Python project).
- version (optional, default=the version tag linked to the commit): version of the package you want to upload.
- build_workflow (optional, default=""): If you have built your package using a job in a _different_ workflow file you will need to give its name here (e.g. `pre-release.yml`).
- environment (optional, default="pre-release"): GitHub environment in which secrets are stored.
Environments help to ensure that only certain operations are available to different user types.
E.g., releasing packages can be given an extra layer of security whereby a maintainer has to approve an action before it can run.
- destination (optional, default="pypi"): One of "pypi" or "internal", to specify what the ultimate destination of the package will be.
If `internal`, the package will be uploaded to <https://packages.arup.com>.
If `pypi`, the package will be uploaded to <https://test.pypi.org/> for testing and <https://pypi.org/> for final upload.

_Required secrets_: `PYPI_API_TOKEN` stored in a GitHub actions environment of the same name as `environment`.
If `destination=internal`, this secret must still be defined, but can be a placeholder string (e.g. "NA").

> [!NOTE]
> To use this action with `destination=internal`, you must request access to the `packages` self-hosted runner for your repository via an Arup service-now request.

### Deploy documentation

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/docs-deploy.yml`

_description_: Deploy [MkDocs](https://www.mkdocs.org/) documentation using [mike](https://github.com/jimporter/mike) to your repository's `gh-pages` branch.

_Inputs_:

- package_name: Name of your package, as defined in your `pyproject.toml` or `setup.py` file (if your repo is a Python project).
- development_version_name (optional, default="develop"): The name of the docs version which follows the project's `main` branch builds, i.e., not linked to a versioned release.
- deploy_type: What type of doc deployment to undertake, option of: ["test", "update_latest", "update_stable"]
   `test` will not deploy any documentation, only dry-run the doc build pipeline to check there are no errors.
   `update_latest` will build the docs and use it to update the `develop` version of your `gh-pages` branch, assuming the alias `latest` links to the named version `develop`.
   `update_stable` will build the docs and use it to add a new version of your docs on `gh-pages` branch and will update the alias `stable` to point at this version.
- notebook_kernel: If jupyter notebooks are included in the docs, specify the kernel name they expect, e.g. the package name.
- py3version (optional, default="11"): Minor version of Python version 3 to run the test on (e.g. `11` for python v3.11).
- additional_env_create_args (optional, default=""): Any additional arguments to pass to conda when creating the python environment.

_Required secrets_: None

### Check accessibility of documentation

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/docs-a11y.yml`

_description_: Check accessibility (a11y) of built documentation using [pa11y-ci](https://github.com/pa11y/pa11y-ci).

_Inputs_:

- upload_report (optional, default=true): If true, upload a workflow artifact containing a full HTML accessibility report.
- allow_pr_comment (optional, default=true): If true, allow a bot to leave a PR comment with a summary of the accessibility report (including a link to the HTML report if `upload_report` is _true_).
- notebook_kernel: If jupyter notebooks are included in the docs, specify the kernel name they expect, e.g. the package name.
- py3version (optional, default="11"): Minor version of Python version 3 to run the test on (e.g. `11` for python v3.11).
- additional_env_create_args (optional, default=""): Any additional arguments to pass to conda when creating the python environment.

_Required secrets_: None

### Run tests on python package

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/python-install-lint-test.yml`

_description_: Run your tests using [pytest](https://docs.pytest.org), (optionally) check your code quality with [Ruff](https://beta.ruff.rs/docs/), and (optionally) upload your test coverage report to [codecov](https://about.codecov.io/).

_Inputs_:

- os: Operating system to run this workflow on. Should match a valid Github runner name.
- py3version: Minor version of Python version 3 to run the test on (e.g. `11` for python v3.11).
- env_name (optional, default={{inputs.os}}-3{{inputs.py3version}}): Name of the Conda environment. If it matches a name of a cached environment in the caller repository, that cache will be used.
- additional_env_create_args (optional, default=""): Any additional arguments to pass to conda when creating the python environment.
- cache_env (optional, default=true): If true, cache the conda environment for speedier CI. Caches use the env name + a hash of the passed arguments. NOTE: this can lead to large amounts of cache space being used (600MB per env)
- notebook_kernel (optional, default=""): If jupyter notebooks are tested, specify the kernel name they expect, e.g. the package name
- lint (optional, default=true): If true, check code quality with the Ruff linter.
- pytest_args (optional, default=""): Additional arguments to pass to pytest.
- upload_to_codecov (optional, default=false): If true, upload coverage report to codecov. This assumes your repository is public as it does not expect an API key.

_Required secrets_: None

### Run memory profiling tests on python package

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/python-memory-profile.yml`

_description_: Run a subset of your tests marked as "high_mem" using [pytest](https://docs.pytest.org) and [memray](https://bloomberg.github.io/memray/).

_Inputs_:

- py3version: Minor version of Python version 3 to run the test on (e.g. `11` for python v3.11).
- additional_env_create_args (optional, default=""): Any additional arguments to pass to conda when creating the python environment.
- upload_flamegraph (optional, default=False): If True, upload the memory profiling flamegraph as an action artefact, stored for 90 days.

_Required secrets_: None

### Notify about action success / failure on a slack channel

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/slack-notify.yml`

_description_: Have a bot notify you of build success or failure on a Slack feed of your choice.

_Inputs_:

- result: Result of running the caller workflow (e.g., 'success', 'failure', 'skipped').
- channel: Slack channel to which the bot notification is sent.
- message: Sub-string to include in the message, e.g. the name of the "caller" workflow.

_Required secrets_: `SLACK_WEBHOOK`

### Check if project is up-to-date with parent template

_URL_: `arup-group/actions-city-modelling-lab/.github/workflows/template-check.yml`

_description_: If your project was generated using a [cookiecutter](https://github.com/cookiecutter/cookiecutter) template, check whether there are changes to the template that could be pulled into the project.
