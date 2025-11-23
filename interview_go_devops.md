# Go Technical Interview Prep (DevOps Context)

## Core Go Concepts & Topics (Experienced Developer Focus)

### Language & Syntax
- Package structure (`main` vs library packages), module layout
- Go modules: semantic import versioning, vendoring, replace directives
- Visibility rules (exported identifiers), `init` functions
- Variables, constants, `iota`, short declaration nuances
- Control structures (`for` variations, `switch`, `select`)
- Functions, closures, variadic functions, `defer` semantics & pitfalls
- Error handling patterns: sentinel, wrapping, custom types, `errors.Is` / `errors.As`...
- ... other content ... 
- **PDF Generation (Optional)**

Pandoc:
```bash
pandoc interview_go_devops.md -o interview_go_devops.pdf
```
Add TOC:
```bash
pandoc --toc --toc-depth=2 interview_go_devops.md -o interview_go_devops.pdf
```