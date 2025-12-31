# CI/CD Pipeline

CI/CD pipelines automate the process of building, testing, and deploying code changes. A well-designed pipeline ensures code quality, security, and reliable deployments.

## Pipeline Structure

### Standard Stages

```typescript
type PipelineStage = 
  | 'install'      // Install dependencies
  | 'lint'         // Code quality checks
  | 'typecheck'    // TypeScript type checking
  | 'test'         // Run tests
  | 'build'        // Compile/transpile code
  | 'security'     // Security scanning
  | 'deploy'       // Deploy to environment
  | 'verify'       // Post-deployment verification
  | 'notify'       // Send notifications
```

### Stage Order and Dependencies

```yaml
stages:
  - install
  - lint
  - typecheck
  - test
  - build
  - security-scan
  - deploy-staging
  - integration-test
  - deploy-production
  - smoke-test
  - notify
```

## Build Stage

### Node.js Installation

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: 'package-lock.json'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          NODE_ENV: production
```

### Multi-Stage Dockerfile

```dockerfile
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY tsconfig.json ./
COPY src/ ./src/
COPY .env.production .env
RUN npm run build

FROM node:20-alpine AS production

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

ENV NODE_ENV=production
EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1

CMD ["node", "dist/app.js"]
```

## Test Stage

### Matrix Testing

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: ['18.x', '20.x', '22.x']
        operating-system: ['ubuntu-latest']
        include:
          - node-version: '20.x'
            operating-system: 'macos-latest'
          - node-version: '20.x'
            operating-system: 'windows-latest'
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run lint
        run: npm run lint
      
      - name: Run type check
        run: npm run typecheck
      
      - name: Run tests
        run: npm run test:coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
```

### Test Scripts

```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui",
    "test:integration": "vitest run --config vitest.integration.config.ts"
  }
}
```

## Security Scanning

### SAST Scanning

```yaml
jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run ESLint Security
        run: npm run lint:security
      
      - name: Run npm audit
        run: npm audit --production --audit-level=high
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

## Quality Gates

### Required Checks

```yaml
jobs:
  quality-gate:
    runs-on: ubuntu-latest
    outputs:
      passed: ${{ steps.check.outputs.passed }}
    steps:
      - id: check
        run: |
          PASSED=true
          
          COVERAGE=$(cat coverage/coverage-summary.json | jq -r '.total.lines.pct')
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% below 80% threshold"
            PASSED=false
          fi
          
          echo "passed=$PASSED" >> $GITHUB_OUTPUT
```

## Deployment Stage

### Environment-Specific Builds

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Get version
        id: version
        run: echo "version=$(npm pkg get version | tr -d '"')" >> $GITHUB_OUTPUT
      
      - name: Build staging
        if: github.ref == 'refs/heads/develop'
        run: |
          echo "VITE_API_URL=${{ secrets.STAGING_API_URL }}" > .env
          npm run build:staging
      
      - name: Build production
        if: github.ref == 'refs/heads/main'
        run: |
          echo "VITE_API_URL=${{ secrets.PRODUCTION_API_URL }}" > .env
          npm run build:production

  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
      
      - name: Deploy to staging
        run: ./scripts/deploy.sh staging

  deploy-production:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      
      - name: Download build
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: dist/
      
      - name: Deploy to production
        run: ./scripts/deploy.sh production
```

## Pipeline as Code

### GitHub Actions Workflow

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production

env:
  NODE_VERSION: '20'
  CACHE_VERSION: 'v1'

jobs:
  ci:
    name: Continuous Integration
    runs-on: ubuntu-latest
    outputs:
      succeeded: ${{ steps.ci-result.outputs.succeeded }}
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: 'package-lock.json'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run type checker
        run: npm run typecheck
      
      - name: Run tests
        run: npm run test:coverage
      
      - id: ci-result
        run: echo "succeeded=true" >> $GITHUB_OUTPUT

  build:
    needs: ci
    name: Build
    runs-on: ubuntu-latest
    if: needs.ci.outputs.succeeded
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: dist/
          retention-days: 7

  deploy:
    needs: build
    name: Deploy
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || github.event.inputs.environment
    environment: ${{ github.ref == 'refs/heads/main' && 'production' || 'staging' }}
    steps:
      - name: Deploy application
        run: ./scripts/deploy.sh ${{ github.event.inputs.environment || 'staging' }}
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

### GitLab CI Configuration

```yaml
stages:
  - install
  - lint
  - test
  - build
  - security
  - deploy

variables:
  NODE_VERSION: "20"
  NPM_CACHE_DIR: "$CI_PROJECT_DIR/.npm"
  
cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - .npm/
    - node_modules/

install:
  stage: install
  image: node:${NODE_VERSION}
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/

lint:
  stage: lint
  image: node:${NODE_VERSION}
  script:
    - npm run lint
  needs:
    - install

test:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm run test:coverage
  coverage: /All files[^|]*\|[^|]*\s+([\d\.]+)/
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  needs:
    - lint

build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
  needs:
    - test

deploy:
  stage: deploy
  image: node:${NODE_VERSION}
  script:
    - ./scripts/deploy.sh ${DEPLOY_ENV}
  environment:
    name: ${DEPLOY_ENV}
    url: https://${DEPLOY_ENV}.example.com
  only:
    - main
    - develop
  variables:
    DEPLOY_ENV: $CI_COMMIT_REF_NAME == 'main' ? 'production' : 'staging'
```

## Verification and Notifications

### Smoke Tests

```yaml
jobs:
  smoke-test:
    needs: deploy
    runs-on: ubuntu-latest
    steps:
      - name: Wait for deployment
        run: sleep 30
      
      - name: Run smoke tests
        run: |
          curl -f ${{ secrets.HEALTH_ENDPOINT }}/health
          curl -f ${{ secrets.HEALTH_ENDPOINT }}/api/status
```

### Notifications

```yaml
jobs:
  notify:
    needs: [deploy, smoke-test]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Send success notification
        if: needs.deploy.result == 'success' && needs.smoke-test.result == 'success'
        run: |
          curl -X POST -H 'Content-type: application/json' \
            --data '{"text":"Deployed successfully to ${{ github.ref }}"}' \
            ${{ secrets.SLACK_WEBHOOK_URL }}
      
      - name: Send failure notification
        if: needs.deploy.result == 'failure' || needs.smoke-test.result == 'failure'
        run: |
          curl -X POST -H 'Content-type: application/json' \
            --data '{"text":"Deployment failed: ${{ github.ref }}"}' \
            ${{ secrets.SLACK_WEBHOOK_URL }}
```

## Common Anti-Patterns

### Anti-Pattern: Long-Running Single Stage

```typescript
async function badPipeline(): Promise<void> {
  await installDependencies();
  await runLinting();
  await runTests();
  await buildApplication();
  await runSecurityScan();
  await deployToStaging();
  await runIntegrationTests();
  await deployToProduction();
  await sendNotification();
}
```

### Better: Cache and Reuse Artifacts

```typescript
async function goodPipeline(): Promise<void> {
  const artifact = await buildOnce();
  await deploy(artifact, 'staging');
  await verify(artifact, 'staging');
  await deploy(artifact, 'production');
}
```

## AI Guidelines

When working with CI/CD pipelines as an AI assistant:

1. **Understand the Pipeline Flow**: Read through the entire pipeline configuration before making changes. Understand the dependencies between stages.

2. **Preserve Artifact Pass-Through**: Ensure build artifacts are properly passed between jobs using artifact storage mechanisms.

3. **Environment-Specific Configuration**: Use environment variables and secrets appropriately for different deployment targets.

4. **Fail Fast**: Configure quality gates to fail early in the pipeline to save resources and time.

5. **Include Rollback Strategy**: Always consider how to rollback if a deployment fails.

6. **Security Scanning**: Recommend security scanning at appropriate stages without slowing down the pipeline unnecessarily.

7. **Caching Strategy**: Suggest caching for dependencies, build outputs, and Docker layers to speed up pipeline execution.
