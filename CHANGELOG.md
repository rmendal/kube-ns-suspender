# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [3.0.0] - 2025-11-XX

### Breaking Changes

- **Minimum Kubernetes version**: Now requires Kubernetes 1.31 or newer
- **Removed**: Support for `batch/v1beta1` CronJob API (removed in Kubernetes 1.25)
- **Removed**: Support for Kubernetes versions < 1.31

### Added

- Support for Kubernetes 1.31, 1.32, 1.33, 1.34
- Multi-version test matrix in CI/CD (tests all four versions)
- Comprehensive migration guide (MIGRATION.md)
- Compatibility matrix in README

### Changed

- **Updated dependencies**:
  - `k8s.io/client-go`: v0.24.3 → v0.33.5
  - `k8s.io/api`: v0.24.3 → v0.33.5
  - `k8s.io/apimachinery`: v0.24.3 → v0.33.5
  - `github.com/kedacore/keda/v2`: v2.8.1 → v2.18.1
  - Go: 1.18 → 1.24.7
- **Updated test infrastructure**:
  - KinD: v0.11.1 → v0.24.0
  - Test cluster: Kubernetes 1.23.4 → 1.31.4 (default)
- **Removed deprecation notice** from README (project is now actively maintained)

### Removed

- `engine/cronjobBeta.go` - Entire file removed (v1beta1 support)
- All references to `batch/v1beta1` CronJob API
- Support for Kubernetes versions 1.21-1.30

### Migration

See [MIGRATION.md](MIGRATION.md) for detailed upgrade instructions and rollback procedures.

## [2.x.x] - Previous Releases (Archived)

See git history at https://github.com/govirtuo/kube-ns-suspender for v2.x releases (supported Kubernetes 1.21-1.24, archived at v2.6.0).
