## Core Philosophy
- Produce maintainable, testable, evolvable C# code.
- Priority order: Correctness > Maintainability > Performance.
- Apply SOLID principles concretely: single-purpose classes, interface-based dependencies, closed-for-modification design.
- Avoid over-engineering (YAGNI) and duplication (DRY). Keep solutions simple (KISS).

## Before Writing Code
1. Restate the requirement in one sentence.
2. Identify the design pattern or abstraction you'll use and why.
3. Flag any tradeoffs or alternatives considered.

## While Writing Code
- Keep methods under 20 lines. Extract logic into well-named private methods.
- Depend on abstractions (interfaces), not concrete implementations.
- Every public method should be unit-testable without mocking infrastructure.

## Do Not (without explicit approval)
- Add or upgrade NuGet packages.
- Change public APIs or method signatures.
- Use static classes, `static` methods, or singletons.
- Leave `// TODO`, `// FIXME`, or stub implementations.

## Stop and Ask When
- Requirements are ambiguous (inputs, outputs, edge cases unclear).
- Multiple patterns apply and the tradeoff is non-trivial.
- The change affects security, financial logic, or public contracts.

## Response Format
- Begin with: `📋 Guidelines applied: [instruction filename(s)]` when referencing `.github/instructions/`.
- Show only the changed method/class with minimal surrounding context — not full files unless asked.
- End with a one-paragraph summary of what changed and why.

## Instruction Files
Actively consult `.github/instructions/` (code, architecture, testing, etc.) for domain-specific rules.
Prioritize those over generic advice. If instructions conflict, ask before proceeding.
