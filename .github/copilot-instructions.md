# Copilot Cloud Agent Onboarding Guide

## Repository Overview

**Purpose:** Template repository for creating production-ready C# applications using modular monolith architecture with vertical slices. This is a starter template—actual project files will be created by users. The repository provides comprehensive guidelines for SOLID principles, modern C# patterns, and structured project organization.

**Technology Stack:**
- **Language:** C# 13+ (modern language features)
- **Runtime:** .NET 10.x (see `.github/workflows/dotnet.yml` for build version)
- **Architecture:** Modular Monolith with Domain-Driven Design (DDD)
- **Build System:** dotnet CLI (no Visual Studio build tools required)
- **Repository Type:** Template (src/ and tests/ are placeholders)

## Build & Validation Commands

**Trust these commands—they are validated and work on Windows, macOS, and Linux.**

### Setup & Restore
```powershell
dotnet restore              # Always run first. Downloads NuGet dependencies.
```

### Build
```powershell
dotnet build --no-restore --configuration Release
```
- Builds all projects in Release mode
- Requires `dotnet restore` to have been run
- Typical duration: 10-30 seconds depending on project size

### Test
```powershell
dotnet test --no-build --configuration Release --logger GitHubActions
```
- Runs all `**/*.Tests.cs` and `**/*Tests.cs` files
- Use `--logger GitHubActions` for CI environments (formats output for GitHub)
- Requires `dotnet build` to have been run first
- Tests should follow Arrange-Act-Assert (AAA) pattern (see `.github/instructions/testing.instructions.md`)

### Complete CI Pipeline (Local Verification)
Run these in order to replicate the GitHub Actions workflow:
```powershell
dotnet restore
dotnet build --no-restore --configuration Release
dotnet test --no-build --configuration Release --logger GitHubActions
```

### Validation
- All commands exit with code 0 on success
- All tests must pass before opening a pull request
- Use `dotnet build --configuration Debug` locally for faster iteration, but **always validate with Release before committing**

## Project Layout

### Directory Structure
```
.github/
  ├── workflows/              # GitHub Actions CI/CD (dotnet.yml)
  ├── instructions/           # Domain-specific coding guidelines
  │   ├── architecture.instructions.md       # Modular monolith structure
  │   ├── code.instructions.md               # Coding standards, error handling
  │   ├── api-design.instructions.md         # REST conventions
  │   ├── csharp-async-patterns.instructions.md # Async/await best practices
  │   ├── background-jobs.instructions.md    # IHostedService patterns
  │   └── testing.instructions.md            # Testing strategy
  └── copilot-instructions.md        # This file
Solution.slnx                 # Solution file (scaffold only)
src/                          # Application source code (placeholders)
tests/                        # Unit & integration tests (placeholders)
```

### Build Configuration
- **Solution file:** `Solution.slnx` (starter scaffold)
- **NuGet cache:** Cached in GitHub Actions (see `dependabot.yml` for dependency updates)
- **Configuration:** Release mode required for CI validation

## Architecture & Module Structure

The repository follows **modular monolith** with **vertical slices**. Each feature module should contain:

```
FeatureName/
  ├── Application/           # Commands, queries, handlers
  ├── Domain/                # Entities, value objects, events
  ├── Infrastructure/        # EF Core, repositories, external APIs
  ├── Presentation/          # Controllers, DTOs, request/response models
  └── Tests/                 # Unit & integration tests
```

For complete architectural details, see `.github/instructions/architecture.instructions.md`.

## Key Instruction Files

**Always consult these files—they contain domain-specific rules that override generic advice:**

1. **architecture.instructions.md** — Module structure, DDD, dependency injection, shared kernel patterns
2. **code.instructions.md** — Modern C# conventions, nullable reference types, Result<T>/ErrorOr<T> patterns
3. **api-design.instructions.md** — REST conventions, HTTP methods, status codes, versioning
4. **csharp-async-patterns.instructions.md** — Async/await, ConfigureAwait, cancellation tokens, composition
5. **background-jobs.instructions.md** — IHostedService, job state machines, graceful shutdown
6. **testing.instructions.md** — Testing strategy, AAA pattern, mocking, TDD

When making code changes, reference the appropriate instruction file in your commit message format:
```
📋 Guidelines applied: [filename]
```

## Continuous Integration

**GitHub Actions Workflow** (`.github/workflows/dotnet.yml`):
- Runs on: Windows, macOS, Linux (matrix strategy)
- Triggers: Push to any branch
- Steps:
  1. Checkout code
  2. Setup .NET 10.x SDK
  3. Load NuGet cache
  4. Run: `dotnet restore`
  5. Run: `dotnet build --no-restore --configuration Release`
  6. Run: `dotnet test --no-build --configuration Release --logger GitHubActions`

**Pull Request Checklist:**
- [ ] All tests pass locally with Release configuration
- [ ] Code follows guidelines in `.github/instructions/`
- [ ] No debug output or temporary code committed
- [ ] Commit message includes guideline references if applicable

## Quick Reference

| Command | Purpose | Precondition |
|---------|---------|-------------|
| `dotnet restore` | Download dependencies | None |
| `dotnet build --no-restore --configuration Release` | Compile all projects | After restore |
| `dotnet test --no-build --configuration Release --logger GitHubActions` | Run all tests | After build |
| `dotnet build --configuration Debug` | Fast local iteration | After restore |

## Do Not (Without Explicit Approval)
- Add or upgrade NuGet packages (these require approval via PRs)
- Change public APIs or method signatures
- Use static classes, `static` methods, or singletons
- Leave `// TODO`, `// FIXME`, or stub implementations
- Commit secrets or credentials
- Modify `.github/workflows/` without testing locally

## When to Ask for Clarification
- Requirements are ambiguous (inputs, outputs, edge cases unclear)
- Multiple patterns apply and the tradeoff is non-trivial
- The change affects security, financial logic, or public contracts
- Instruction files conflict or are unclear

**Trust the instructions—only search the codebase if instructions are incomplete or incorrect.**
