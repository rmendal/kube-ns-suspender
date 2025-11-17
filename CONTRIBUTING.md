# Contributing to kube-ns-suspender

Thank you for your interest in contributing to kube-ns-suspender! This document provides guidelines and information for contributors.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [How to Contribute](#how-to-contribute)
- [Development Setup](#development-setup)
- [Testing](#testing)
- [Pull Request Process](#pull-request-process)
- [Coding Standards](#coding-standards)
- [Commit Messages](#commit-messages)
- [Reporting Bugs](#reporting-bugs)
- [Suggesting Enhancements](#suggesting-enhancements)

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/). By participating, you are expected to uphold this code. Please report unacceptable behavior by opening an issue.

## Getting Started

- Make sure you have a [GitHub account](https://github.com/signup/free)
- Fork the repository on GitHub
- Read the [README.md](README.md) to understand the project
- Check out the [open issues](https://github.com/adnilim/kube-ns-suspender/issues)

## How to Contribute

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, include:

- **Clear title and description**
- **Steps to reproduce** the issue
- **Expected behavior** vs **actual behavior**
- **Environment details**:
  - Kubernetes version
  - kube-ns-suspender version
  - KEDA version (if applicable)
  - Cloud provider (if relevant)
- **Logs** from kube-ns-suspender pod
- **Manifests** used (sanitize any sensitive data)

### Suggesting Enhancements

Enhancement suggestions are tracked as GitHub issues. When creating an enhancement suggestion, include:

- **Clear title and description** of the proposed feature
- **Use case** explaining why this enhancement would be useful
- **Possible implementation** if you have ideas
- **Alternatives considered**

### Pull Requests

We actively welcome pull requests! Follow these steps:

1. Fork the repo and create your branch from `main`
2. If you've added code that should be tested, add tests
3. If you've changed APIs or behavior, update the documentation
4. Ensure the test suite passes
5. Make sure your code follows the existing style
6. Write a clear pull request description

## Development Setup

### Prerequisites

- Go 1.24 or later
- Docker (for building container images)
- kubectl
- [KinD](https://kind.sigs.k8s.io/) (for local testing)
- make

### Local Development

1. **Clone your fork**:
   ```bash
   git clone https://github.com/YOUR_USERNAME/kube-ns-suspender.git
   cd kube-ns-suspender
   ```

2. **Add upstream remote**:
   ```bash
   git remote add upstream https://github.com/adnilim/kube-ns-suspender.git
   ```

3. **Install dependencies**:
   ```bash
   go mod download
   ```

4. **Build the project**:
   ```bash
   make build
   ```

5. **Set up a local Kubernetes cluster**:
   ```bash
   make kind-start
   export KUBECONFIG=$(pwd)/kubeconfig.yaml
   ```

6. **Run in development mode** (see [README.md](README.md#development-flow)):
   ```bash
   kubectl apply -f manifests/testing-namespace
   devspace dev
   ```

## Testing

### Running Tests

**Unit tests**:
```bash
make test
```

**E2E tests** (requires KinD cluster):
```bash
make kind-start
make e2e
```

**Specific test**:
```bash
go test ./engine -v -run TestDeploymentSuspension
```

### Writing Tests

- Write unit tests for all new functions
- Add E2E tests for new resource types or suspension behavior
- Use table-driven tests where appropriate
- Mock external dependencies (Kubernetes API, AWS API, etc.)

## Pull Request Process

1. **Update documentation** for any user-facing changes
2. **Add tests** covering your changes
3. **Update CHANGELOG.md** following [Keep a Changelog](https://keepachangelog.com/) format
4. **Ensure CI passes** - all tests and linting must pass
5. **Request review** from maintainers
6. **Address feedback** promptly and professionally
7. **Squash commits** if requested before merge

### PR Title Format

Use conventional commit format for PR titles:

- `feat: add support for DaemonSets`
- `fix: resolve CronJob suspension race condition`
- `docs: update installation instructions`
- `test: add E2E tests for StatefulSets`
- `chore: update dependencies`
- `refactor: simplify suspension logic`

## Coding Standards

### Go Code Style

- Follow [Effective Go](https://golang.org/doc/effective_go.html)
- Use `gofmt` for formatting
- Use `golangci-lint` for linting
- Write clear, self-documenting code
- Add comments for exported functions and types
- Keep functions small and focused
- Avoid global variables

### Project-Specific Guidelines

- Use structured logging with `zerolog`
- Follow existing patterns for Kubernetes client usage
- Respect namespace isolation
- Handle errors gracefully
- Use context for cancellation and timeouts
- Add metrics for observable behavior

### Example Code Style

```go
// Good: Clear, documented, error handling
// SuspendDeployment scales a deployment to zero replicas and stores the original count.
func (e *Engine) SuspendDeployment(ctx context.Context, namespace, name string) error {
    deployment, err := e.ClientSet.AppsV1().Deployments(namespace).Get(ctx, name, metav1.GetOptions{})
    if err != nil {
        return fmt.Errorf("failed to get deployment %s/%s: %w", namespace, name, err)
    }

    // Store original replica count
    originalReplicas := *deployment.Spec.Replicas
    if deployment.Annotations == nil {
        deployment.Annotations = make(map[string]string)
    }
    deployment.Annotations[originalReplicasAnnotation] = strconv.Itoa(int(originalReplicas))

    // Scale to zero
    zero := int32(0)
    deployment.Spec.Replicas = &zero

    _, err = e.ClientSet.AppsV1().Deployments(namespace).Update(ctx, deployment, metav1.UpdateOptions{})
    if err != nil {
        return fmt.Errorf("failed to suspend deployment %s/%s: %w", namespace, name, err)
    }

    e.Logger.Info().
        Str("namespace", namespace).
        Str("deployment", name).
        Int32("original_replicas", originalReplicas).
        Msg("Deployment suspended")

    return nil
}
```

## Commit Messages

Write clear, descriptive commit messages following these guidelines:

### Format

```
<type>: <subject>

<body>

<footer>
```

### Type

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `test`: Adding or updating tests
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `chore`: Maintenance tasks
- `ci`: CI/CD changes

### Example

```
feat: add support for HorizontalPodAutoscaler suspension

Add ability to suspend HorizontalPodAutoscalers by setting minReplicas
and maxReplicas to 0 when a namespace is suspended. Store original
values in annotations for restoration.

Closes #42
```

## Release Process

Releases are handled by maintainers. If you're interested in becoming a maintainer, please reach out by opening an issue.

### Versioning

This project follows [Semantic Versioning](https://semver.org/):

- **MAJOR** version for incompatible API changes or Kubernetes version drops
- **MINOR** version for new functionality in a backwards compatible manner
- **PATCH** version for backwards compatible bug fixes

## Questions?

If you have questions about contributing, feel free to:

- Open an [issue](https://github.com/adnilim/kube-ns-suspender/issues/new) with the `question` label
- Start a [discussion](https://github.com/adnilim/kube-ns-suspender/discussions) (if enabled)

## License

By contributing to kube-ns-suspender, you agree that your contributions will be licensed under the MIT License.

## Attribution

Thank you to all contributors who help make this project better! Contributors will be recognized in release notes and can be viewed in the [GitHub contributors graph](https://github.com/adnilim/kube-ns-suspender/graphs/contributors).

---

**Happy Contributing!** 🎉
