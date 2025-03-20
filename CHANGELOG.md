<!---
Changelog headings can be any of:

Added: for new features.
Changed: for changes in existing functionality.
Deprecated: for soon-to-be removed features.
Removed: for now removed features.
Fixed: for any bug fixes.
Security: in case of vulnerabilities.
-->

# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Fixed

- Move to using `conda` to install the test environment, to mitigate issues arising from accessing `mamba` environments on Windows machines (#69).
- Mamba environment creation composite action uses `mamba create --yes` instead of `mamba update` to catch unpinned dependencies (#28).
- Pa11y CI failing due to old Node.js version. Fixed by updating from v16 to v20.
- `pipbuild` mamba environment, to use the `python-build` package instead of the equivalent archived `build` package (#45).
- `setup-miniconda` pinned to < v3.1 to avoid bug when running on Windows: <https://github.com/conda-incubator/setup-miniconda/issues/371>

### Changed

- Updated `conda-incubator/setup-miniconda` to v3.1.1, switching from using `mamba` to `conda` in the process (#69).
- Loosened `memray` dependency pinning in `python-memory-profile.yml` to ensure version compatibility with a wider range of project pinning.
- Updated environment caching method to only cache environment lockfiles / `explicit` lists, if using `conda-incubator/setup-miniconda`, to reduce Windows runner build times at the expense of slower Linux/OSX build times (#30).
- Moved to `conda-incubator/setup-miniconda` instead of `mamba-org/setup-micromamba` where we would benefit from having `mamba`/`conda` available on the runner PATH (#26).
- Added an optional zip file name parameter to `aws-upload/yml` (#57).

### Added

- Ability to upload pip / conda packages to <https://packages.arup.com> for internal Arup projects.
- Composite action for building a project-specific conda environment, used across reusable workflows but also available for direct use as a step in other projects (#26).
- Environment cache directory within the runner working directory (`.cache/envs`) (#29).

## [v1.0.0] - 2024-07-12

### Fixed

### Added

- README

- Reusable workflows for:
  - Upload package to AWS
  - Build a conda package
  - Upload a conda package
  - Deploy documentation
  - Run tests on python package
  - Run memory profiling tests on python package
  - Notify about action success / failure on a slack channel
  - Check whether project and parent template have diverged
  - Check accessibility of documentation

- GitHub Action to validate the reusable workflows themselves
- Optional `image-tags` input parameter added to the AWS Upload workflow (#19)
- Optional `additional_env_create_args` input parameter added to the docs deployment workflow (#25)

### Changed

### Removed
