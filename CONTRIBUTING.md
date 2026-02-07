# Contributing to @monetra/core

First off, thanks for taking the time to contribute! `@monetra/core` is a precision money library, and we value correctness above all else.

## How to Contribute

1.  **Fork the repository**
2.  **Checkout the target version branch**
    *   Identify the version you are working on (e.g., `v0.1.0`).
    *   Checkout that branch from your fork.
    *   *Do not target `main` or `dev` unless specifically instructed.*
3.  **Create your feature branch** from that version branch (`git checkout -b feature/amazing-feature`)
4.  **Commit your changes** (`git commit -m 'feat: add amazing feature'`)
5.  **Push to the branch** (`git push origin feature/amazing-feature`)
6.  **Open a Pull Request** targeting the version branch you started from.

## Guidelines

### Issue Tracking & Roadmap

We use [GitHub Projects](https://github.com/zugobite/monetra.core) to track our roadmap and active development.

- **Before starting work**: Please check the "In Progress" column to ensure no one else is working on the same task.
- **Picking up an issue**: Comment on the issue asking to be assigned, so we can move it to "In Progress".
- **Status Updates**: Check the board for the latest status on features and bugs.

### Code Style

- We use **TypeScript** with `strict: true`.
- No floating-point arithmetic is allowed.
- All operations must be immutable.
- Run `pnpm run lint` to ensure code style compliance.

### Testing

- **Correctness and Rigor are paramount.**
- Since `@monetra/core` handles monetary values, we uphold strict standards for testing and reliability.
- All new features must include comprehensive unit tests.
- **Work will not be merged if it does not pass all coverage tests.**
- Run `pnpm test` to verify your changes.
- Run `pnpm test:coverage` to check code coverage.
- Ensure 100% test coverage for new logic.

### Documentation

- Update `README.md` or `docs/` if you change public APIs.
- Add TSDoc comments to all exported functions and classes.

### Commits

- Keep commits atomic and well-described.
- We follow [Conventional Commits](https://www.conventionalcommits.org/) (e.g., `feat: add allocation`, `fix: rounding error`).

## Development Workflow

```bash
# Install dependencies
pnpm install

# Run tests
pnpm test

# Build the package
pnpm run build

# Format code
pnpm run format
```
