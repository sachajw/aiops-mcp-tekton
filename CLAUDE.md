# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tekton MCP Server - A Model Context Protocol server that provides AI models with tools to interact with Tekton CI/CD pipelines on Kubernetes clusters. Currently focuses on `tektoncd/pipeline` objects (Pipelines, Tasks, PipelineRuns, TaskRuns, StepActions).

## Common Commands

```bash
# Build
go build -v ./...

# Run all tests with race detection
go test -v -race -timeout 5m ./...

# Run a single test
go test -v -race -timeout 5m ./internal/tools -run TestList

# Lint (uses golangci-lint v2 config)
golangci-lint run --timeout=10m

# Format code
gofmt -w .

# Deploy to local Kubernetes cluster (installs Tekton + MCP server)
make deploy_tekton

# Deploy only the MCP server (assumes Tekton already installed)
make apply

# Remove from cluster
make clean_tekton
```

## Architecture

### Entry Point
- `cmd/tekton-mcp-server/main.go` - Server initialization with HTTP (`:8080`) or stdio transport modes. Uses Knative injection for Kubernetes client setup.

### Core Packages
- `internal/tools/` - MCP tool implementations (list, get, create, update, delete, start, restart operations for Tekton resources)
- `internal/resources/` - MCP Resource handlers using `tekton://{resourceType}/{namespace}/{name}` URI template
- `internal/version/` - Version management

### Testing Pattern
Tests use Tekton's fake context framework:
```go
ctx, _ := ttesting.SetupFakeContext(t)
data := test.Data{Pipelines: [...], Tasks: [...]}
test.SeedTestData(t, ctx, data)
ss, cs := newSession(t, ctx)  // Creates in-memory MCP server/client
```

### Kubernetes Deployment
Config in `config/` directory:
- Namespace: `tekton-mcp`
- Service account with RBAC for Tekton resources
- Deployment runs as non-root with read-only filesystem

## Code Conventions

### Linting Rules (from .golangci.yml)
- Use `sigs.k8s.io/yaml` instead of `github.com/ghodss/yaml`
- Avoid `io/ioutil` (use `io` and `os` packages)
- 70+ linters enabled; run `golangci-lint run` before committing

### Dependencies
- MCP SDK: `github.com/modelcontextprotocol/go-sdk`
- Tekton: `github.com/tektoncd/pipeline`
- Kubernetes clients: `k8s.io/client-go`
- Knative injection: `knative.dev/pkg`

### Container Build
Uses `ko` for building Go container images with `gcr.io/distroless/static:nonroot` base image.
