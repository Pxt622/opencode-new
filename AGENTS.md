# AGENTS.md

This file provides critical instructions for agents working in this repository. Only include information an agent would miss without explicit guidance.

## Key Commands

- **Run all tests**: `npm run test`
- **Run browser tests**: `npm run test-browser`
- **Run node tests**: `npm run test-node`
- **Build project**: `npm run compile`
- **Watch changes**: `npm run watch`
- **Start development server**: `npm run electron`
- **Lint code**: `npm run eslint`
- **Style check**: `npm run stylelint`
- **Run smoke tests**: `npm run smoketest`

## Development Workflow

1. Use `npm install` to set up dependencies
2. Run `npm run build` before committing
3. Use `npm run test` for full test suite
4. For browser testing: `npm run test-browser`
5. For node-specific tests: `npm run test-node`

## Notes
- The `watch` script uses `deemon` for process management
- Browser tests require Playwright: `npm install playwright` first
- Use `npm run test-browser-no-install` for quick browser tests
- The `compile` script handles TypeScript compilation
- `npm run watch` is recommended for development
- Use `npm run lint` for code quality checks
- For CI: `npm run core-ci` and `npm run extensions-ci`
- 全局规则：始终使用中文进行输出