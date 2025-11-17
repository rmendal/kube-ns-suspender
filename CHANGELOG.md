# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [3.0.5] - 2025-11-17

### Fixed

- **Critical**: Container tags now use **newline-separated format** instead of comma-separated
  - `docker/build-push-action` requires tags to be newline-separated (one per line)
  - Previous comma-separated format was being ignored, causing only first tag to be used
- Use heredoc (EOF) to properly preserve newlines in GitHub Actions outputs
- Tags now correctly generated: `3.0.5`, `3.0`, `3`, `latest` (all without 'v' prefix)

### Technical Details

The root cause was tag format:
- ❌ Wrong: `tag1,tag2,tag3` (comma-separated)
- ✅ Correct: Multi-line format (newline-separated)

## [3.0.4] - 2025-11-17

### Fixed

- **Tag generation**: Fixed bash variable expansion in GitHub Actions workflow
  - Use `${{ github.ref }}` instead of `$GITHUB_REF` environment variable
  - Use `${{ env.IMAGE_NAME }}` instead of `$IMAGE_NAME` for consistency
- **Debugging**: Added debug step to show exactly which tags will be pushed

### Changed

- Improved logging in tag generation step for better troubleshooting

## [3.0.3] - 2025-11-17

### Fixed

- **Container image tagging**: Replaced `docker/metadata-action` with manual tag generation to ensure all semantic version tags are created correctly
- Container images now properly tagged with `X.Y.Z`, `X.Y`, `X`, and `latest` (without 'v' prefix)
- Previous releases (v3.0.1, v3.0.2) only created `latest` and `vX.Y.Z` tags due to metadata-action configuration issues

### Changed

- Manual tag generation in workflow for more reliable and predictable tagging

## [3.0.2] - 2025-11-17

### Fixed

- **Container image tagging**: Fixed semver tag generation to properly create `X.Y.Z`, `X.Y`, `X` tags without 'v' prefix
- **CI/CD workflow**: Tag pushes now always trigger the full workflow (build, test, publish), even when only non-code files changed
- Previously, pushing a tag like `v3.0.1` after only changing CHANGELOG/workflow files would not trigger image builds

### Changed

- Simplified `docker/metadata-action` configuration for more reliable tag generation

## [3.0.1] - 2025-11-17

### Added

- **Multi-architecture container images**: Images now built for both `linux/amd64` and `linux/arm64` platforms
- Support for Apple Silicon (M1/M2/M3) and other ARM64 systems

### Changed

- **Container image tagging**: Semantic version tags now use version numbers without 'v' prefix
  - Git tags: `v3.0.1` (with 'v')
  - Container tags: `3.0.1`, `3.0`, `3`, `latest` (without 'v')
- Updated GitHub Actions workflow to latest versions (v3-v5)
- Added build caching for faster CI/CD builds
- Updated `docker/build-push-action` to v5 with QEMU for cross-platform builds

## [3.0.0] - 2025-11-17

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
