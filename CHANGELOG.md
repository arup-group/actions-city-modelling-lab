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

- GitHub Action to validate the reusable workflows themselves
- Optional `image-tags` input parameter added to the AWS Upload workflow ([#19](https://github.com/arup-group/actions-city-modelling-lab/issues/19))

### Changed

### Removed