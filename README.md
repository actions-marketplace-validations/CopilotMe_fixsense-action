# FixSense — AI Test Failure Analysis

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-FixSense-green?logo=github)](https://github.com/marketplace/actions/fixsense-ai-test-failure-analysis)

Analyze CI test failures with AI. Detect **app bugs vs test bugs**, get root cause analysis and actionable fix suggestions — directly in your PR.

## Features

- **Root Cause Analysis** — AI explains why your test failed in 2-4 sentences
- **App Bug vs Test Bug** — Distinguishes application bugs from test assertion errors
- **Flakiness Score** — 0-100 score indicating if the failure is likely flaky
- **Fix Suggestions** — Actionable steps to fix the failing test
- **PR Comments** — Automatic analysis posted as a PR comment

## Quick Start

Add to your workflow **after** your test step:

```yaml
- name: Run tests
  run: npx playwright test
  continue-on-error: true

- name: Analyze failures
  if: failure() || steps.tests.outcome == 'failure'
  uses: CopilotMe/fixsense-action@v1
  with:
    api-key: ${{ secrets.FIXSENSE_API_KEY }}
```

## Full Example

```yaml
name: Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci
      - run: npx playwright install --with-deps

      - name: Run Playwright tests
        id: tests
        run: npx playwright test
        continue-on-error: true

      - name: FixSense Analysis
        if: steps.tests.outcome == 'failure'
        uses: CopilotMe/fixsense-action@v1
        with:
          api-key: ${{ secrets.FIXSENSE_API_KEY }}
          results-path: test-results/

      - uses: actions/upload-artifact@v4
        if: always()
        with:
          name: test-results
          path: test-results/
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | Yes | — | FixSense API key ([get one](https://fix-sense.vercel.app/settings)) |
| `results-path` | No | `test-results/` | Path to JUnit XML test results |
| `post-comment` | No | `true` | Post analysis as PR comment |
| `api-url` | No | `https://fix-sense.vercel.app` | API URL (for self-hosted) |

## Outputs

| Output | Description |
|--------|-------------|
| `analyses` | JSON array of analysis results |
| `failure-count` | Number of failures analyzed |

## Supported Frameworks

Any framework that produces **JUnit XML** reports:

- **Playwright** — `reporter: [['junit', { outputFile: 'test-results/junit.xml' }]]`
- **Cypress** — `cypress-junit-reporter`
- **Jest** — `jest-junit`
- **pytest** — `--junitxml=test-results/junit.xml`
- **JUnit/TestNG** — native XML output

## How It Works

1. Your tests run and produce JUnit XML results
2. FixSense parses the XML for failures
3. Each failure is sent to the FixSense AI for analysis
4. Results are posted as a PR comment with root cause, fix suggestions, and flakiness score

## PR Comment Preview

The action posts a table with all failures and expandable details:

| Test | Root Cause | Flakiness | Confidence |
|------|-----------|-----------|------------|
| `login.spec.ts > should login` | Timeout waiting for selector `.submit-btn` | 🟡 65 | 🟢 high |

## Get Your API Key

1. Go to [fix-sense.vercel.app](https://fix-sense.vercel.app)
2. Sign in with GitHub
3. Copy your API key from Settings
4. Add it as a repository secret: `FIXSENSE_API_KEY`

## Links

- [FixSense Dashboard](https://fix-sense.vercel.app)
- [Documentation](https://fix-sense.vercel.app/docs)

## License

MIT
