# Deployment

A comprehensive guide to deployment practices, CI/CD pipelines, environment management, and release strategies for reliable software delivery.

## Table of Contents

1. [CI/CD Pipeline](#cicd-pipeline)
2. [Staging and Production Environments](#staging-and-production-environments)
3. [Deployment Strategies](#deployment-strategies)
4. [Configuration Management](#configuration-management)
5. [Secret Management](#secret-management)
6. [Monitoring and Alerting](#monitoring-and-alerting)
7. [Rollback Procedures](#rollback-procedures)
8. [Git Workflow and Version Control](#git-workflow-and-version-control)
9. [AI-Assisted Deployment Guidelines](#ai-assisted-deployment-guidelines)

---

## CI/CD Pipeline

CI/CD pipelines automate the process of building, testing, and deploying code changes. A well-designed pipeline ensures code quality, security, and reliable deployments.

### Pipeline Structure

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

### Build Stage

#### Node.js Installation

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

#### Multi-Stage Dockerfile

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

### Test Stage

#### Matrix Testing

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

#### Test Scripts

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

### Security Scanning

#### SAST Scanning

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

### Quality Gates

#### Required Checks

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

### Deployment Stage

#### Environment-Specific Builds

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

### Pipeline as Code

#### GitHub Actions Workflow

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

#### GitLab CI Configuration

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

### Verification and Notifications

#### Smoke Tests

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

#### Notifications

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

---

## Staging and Production Environments

Proper environment management is essential for reliable software delivery. Staging and production environments serve distinct purposes in the deployment lifecycle.

### Environment Overview

#### Environment Comparison

```typescript
interface EnvironmentConfig {
  name: string;
  purpose: string;
  dataRealism: 'production-mirror' | 'synthetic' | 'sample';
  accessLevel: 'developers' | 'qa' | 'stakeholders' | 'public';
  isolation: 'shared' | 'dedicated';
  scale: 'minimal' | 'production-scaled' | 'production-mirror';
  refreshFrequency: 'continuous' | 'daily' | 'weekly' | 'manual';
}

const environments: Record<string, EnvironmentConfig> = {
  development: {
    name: 'Development',
    purpose: 'Local development and rapid iteration',
    dataRealism: 'synthetic',
    accessLevel: 'developers',
    isolation: 'shared',
    scale: 'minimal',
    refreshFrequency: 'continuous',
  },
  staging: {
    name: 'Staging',
    purpose: 'Pre-production validation and testing',
    dataRealism: 'anonymized-production',
    accessLevel: 'developers',
    isolation: 'dedicated',
    scale: 'production-scaled',
    refreshFrequency: 'daily',
  },
  production: {
    name: 'Production',
    purpose: 'Live user-facing application',
    dataRealism: 'production',
    accessLevel: 'public',
    isolation: 'dedicated',
    scale: 'production-mirror',
    refreshFrequency: 'manual',
  },
};
```

### Staging Environment

#### Purpose and Goals

```typescript
interface StagingGoals {
  validation: string[];
  testing: string[];
  performance: string[];
  integration: string[];
}

const stagingGoals: StagingGoals = {
  validation: [
    'Verify build artifacts work correctly',
    'Confirm environment configuration is correct',
    'Test deployment scripts and procedures',
  ],
  testing: [
    'QA manual testing',
    'User acceptance testing',
    'Exploratory testing',
  ],
  performance: [
    'Baseline performance metrics',
    'Load testing under production-like conditions',
    'Stress testing to breaking point',
  ],
  integration: [
    'Third-party service integration',
    'Database migrations',
    'API contract verification',
  ],
};
```

#### Staging Setup

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: production
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=staging
      - DATABASE_URL=postgresql://user:pass@staging-db:5432/app
      - REDIS_URL=redis://staging-redis:6379
      - API_URL=https://api.staging.example.com
      - FEATURE_FLAGS_ENABLED=true
      - LOG_LEVEL=debug
    depends_on:
      staging-db:
        condition: service_healthy
      staging-redis:
        condition: service_started

  staging-db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    volumes:
      - postgres_staging_data:/var/lib/postgresql/data
      - ./seed-data:/docker-entrypoint-initdb.d
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 10s
      timeout: 5s
      retries: 5

  staging-redis:
    image: redis:7-alpine
    volumes:
      - redis_staging_data:/data

volumes:
  postgres_staging_data:
  redis_staging_data:
```

#### Data Management

```typescript
interface StagingDataStrategy {
  source: 'production-mirror' | 'synthetic' | 'hybrid';
  anonymization: boolean;
  refreshSchedule: string;
  sizeLimits: DataSizeLimits;
}

interface DataSizeLimits {
  maxRecords: number;
  maxStorage: string;
  retentionDays: number;
}

async function refreshStagingData(
  strategy: StagingDataStrategy
): Promise<void> {
  if (strategy.source === 'production-mirror') {
    await anonymizeAndCopyProductionData();
  }
  
  if (strategy.anonymization) {
    await applyAnonymizationRules();
  }
  
  await applyDataSampling(strategy.sizeLimits);
  await updateSearchIndexes();
  await warmCaches();
}

async function anonymizeAndCopyProductionData(): Promise<void> {
  const snapshot = await createDatabaseSnapshot('production');
  
  await restoreDatabase(snapshot, 'staging');
  
  await Promise.all([
    anonymizeTable('users', {
      email: 'email',
      name: 'name',
      phone: 'phone',
      address: 'address',
    }),
    anonymizeTable('orders', {
      creditCard: 'credit_card',
      customerName: 'customer_name',
    }),
  ]);
}
```

### Production Environment

#### Production Readiness

```typescript
interface ProductionReadinessChecklist {
  infrastructure: string[];
  security: string[];
  monitoring: string[];
  operations: string[];
  documentation: string[];
}

const productionReadiness: ProductionReadinessChecklist = {
  infrastructure: [
    'Load balancers configured with health checks',
    'Auto-scaling policies defined',
    'Multi-AZ deployment for high availability',
    'Database replication and backups configured',
    'CDN configured for static assets',
  ],
  security: [
    'HTTPS/TLS configured with valid certificates',
    'Secret management (Vault/AWS Secrets Manager)',
    'WAF rules enabled',
    'DDoS protection enabled',
    'IP allowlisting for admin endpoints',
  ],
  monitoring: [
    'Application performance monitoring (APM)',
    'Error tracking and alerting',
    'Log aggregation and analysis',
    'Uptime monitoring',
    'Custom dashboards for key metrics',
  ],
  operations: [
    'Runbooks documented for common issues',
    'On-call rotation established',
    'Escalation procedures defined',
    'Incident response process documented',
    'Rollback procedure tested',
  ],
  documentation: [
    'Architecture diagrams updated',
    'API documentation current',
    'Environment variables documented',
    'Deployment procedures verified',
    'Disaster recovery plan documented',
  ],
};
```

---

## Deployment Strategies

### Blue-Green Deployment

```typescript
interface BlueGreenConfig {
  blueEnv: string;
  greenEnv: string;
  loadBalancerArn: string;
  trafficSwitchRatio: number;
  healthCheckPath: string;
  verificationTimeout: number;
}

async function executeBlueGreenDeployment(
  config: BlueGreenConfig,
  newVersion: string
): Promise<DeploymentResult> {
  const { blueEnv, greenEnv, loadBalancerArn } = config;
  
  console.log(`Deploying ${newVersion} to ${greenEnv}`);
  await deployToEnvironment(greenEnv, newVersion);
  
  const isGreenHealthy = await verifyEnvironmentHealth(
    greenEnv,
    config.healthCheckPath,
    config.verificationTimeout
  );
  
  if (!isGreenHealthy) {
    await rollbackToBlueGreen(blueEnv, greenEnv);
    return { success: false, reason: 'Green environment unhealthy' };
  }
  
  console.log('Switching traffic to green environment');
  await updateLoadBalancerTarget(
    loadBalancerArn,
    greenEnv,
    config.trafficSwitchRatio
  );
  
  const trafficSwitchSuccess = await monitorTrafficShift(
    blueEnv,
    greenEnv,
    5 * 60 * 1000
  );
  
  if (!trafficSwitchSuccess) {
    console.log('Traffic switch issue detected, initiating rollback');
    await rollbackToBlueGreen(greenEnv, blueEnv);
    return { success: false, reason: 'Traffic shift failed' };
  }
  
  await promoteEnvironment(blueEnv, greenEnv);
  
  return { success: true, version: newVersion };
}
```

### Canary Deployment

```typescript
interface CanaryConfig {
  baselineEnv: string;
  canaryPercentage: number;
  rolloutSteps: number[];
  metricsToMonitor: string[];
  failureThreshold: number;
  autoRollback: boolean;
}

async function executeCanaryDeployment(
  config: CanaryConfig,
  newVersion: string
): Promise<CanaryResult> {
  const { baselineEnv, canaryPercentage, rolloutSteps } = config;
  
  console.log(`Deploying ${newVersion} with ${canaryPercentage}% traffic`);
  await deployToCanaryInstances(newVersion, canaryPercentage);
  
  for (const step of rolloutSteps) {
    const newPercentage = step;
    console.log(`Increasing canary traffic to ${newPercentage}%`);
    
    await updateTrafficSplit(baselineEnv, newPercentage);
    
    const metrics = await collectMetrics(
      config.metricsToMonitor,
      2 * 60 * 1000
    );
    
    const shouldContinue = evaluateCanaryMetrics(
      metrics,
      config.failureThreshold
    );
    
    if (!shouldContinue) {
      console.log('Canary metrics indicate problems');
      
      if (config.autoRollback) {
        await rollbackCanary();
      }
      
      return {
        success: false,
        stoppedAtPercentage: step,
        reason: 'Metrics exceeded failure threshold',
      };
    }
  }
  
  await updateTrafficSplit(baselineEnv, 100);
  
  return { success: true, finalVersion: newVersion };
}
```

### Rolling Deployment

```typescript
interface RollingDeploymentConfig {
  environment: string;
  batchSize: number;
  healthCheckPath: string;
  batchPauseSeconds: number;
  maxFailurePercentage: number;
}

async function executeRollingDeployment(
  config: RollingDeploymentConfig,
  newVersion: string
): Promise<RollingDeploymentResult> {
  const instances = await getInstanceIds(config.environment);
  const totalInstances = instances.length;
  const batchSize = Math.min(config.batchSize, totalInstances);
  const batches = chunkArray(instances, batchSize);
  
  let failedCount = 0;
  const failedInstances: string[] = [];
  
  for (let i = 0; i < batches.length; i++) {
    const batch = batches[i];
    console.log(`Deploying batch ${i + 1}/${batches.length}`);
    
    await Promise.all(
      batch.map(instanceId => updateInstance(instanceId, newVersion))
    );
    
    await waitForInstancesHealthy(batch, config.healthCheckPath);
    
    const batchFailures = await checkInstanceHealth(batch);
    failedCount += batchFailures.length;
    failedInstances.push(...batchFailures);
    
    const failureRate = failedCount / totalInstances;
    if (failureRate > config.maxFailurePercentage) {
      await rollbackBatches(instances, failedInstances);
      return {
        success: false,
        deployedBatches: i,
        failedInstances,
        failureRate,
      };
    }
    
    if (i < batches.length - 1) {
      await sleep(config.batchPauseSeconds * 1000);
    }
  }
  
  return {
    success: true,
    totalInstances,
    failedInstances,
    failureRate: failedCount / totalInstances,
  };
}
```

---

## Configuration Management

### Configuration Management

```typescript
interface EnvironmentVariables {
  app: Record<string, string>;
  database: Record<string, string>;
  externalServices: Record<string, string>;
  featureFlags: Record<string, boolean>;
}

function generateEnvironmentConfig(
  env: 'staging' | 'production'
): EnvironmentVariables {
  const baseConfig = {
    app: {
      NODE_ENV: env,
      LOG_LEVEL: env === 'production' ? 'info' : 'debug',
      PORT: process.env.PORT || '3000',
      API_VERSION: 'v1',
    },
    database: {
      DATABASE_POOL_SIZE: env === 'production' ? '20' : '5',
      DATABASE_CONNECTION_TIMEOUT: '30000',
      DATABASE_IDLE_TIMEOUT: '600000',
    },
    externalServices: {
      PAYMENT_GATEWAY_URL: getSecret(`payment-gateway-${env}`),
      EMAIL_SERVICE_URL: getSecret(`email-service-${env}`),
      ANALYTICS_ENDPOINT: getSecret(`analytics-endpoint-${env}`),
    },
    featureFlags: {
      NEW_DASHBOARD: env === 'staging',
      BETA_FEATURES: env === 'staging',
      EXPERIMENTAL_API: false,
    },
  };
  
  if (env === 'production') {
    return {
      ...baseConfig,
      app: {
        ...baseConfig.app,
        LOG_LEVEL: 'info',
        ENABLE_DEBUG_ENDPOINTS: false,
      },
      featureFlags: {
        ...baseConfig.featureFlags,
        NEW_DASHBOARD: false,
        BETA_FEATURES: false,
      },
    };
  }
  
  return baseConfig;
}
```

### Environment-Specific Settings

```typescript
// config/environments/development.ts
export const developmentConfig = {
  database: {
    host: 'localhost',
    port: 5432,
    ssl: false
  },
  email: {
    provider: 'console',
    from: 'noreply@localhost'
  },
  cache: {
    provider: 'memory'
  }
};

// config/environments/production.ts
export const productionConfig = {
  database: {
    host: process.env.DATABASE_HOST,
    port: parseInt(process.env.DATABASE_PORT),
    ssl: true
  },
  email: {
    provider: 'ses',
    from: process.env.EMAIL_FROM
  },
  cache: {
    provider: 'redis',
    url: process.env.REDIS_URL
  }
};

// config/index.ts
const env = process.env.NODE_ENV || 'development';

export const config = env === 'production'
  ? productionConfig
  : developmentConfig;
```

---

## Secret Management

```typescript
interface SecretProvider {
  getSecret(name: string): Promise<string>;
  setSecret(name: string, value: string): Promise<void>;
  rotateSecret(name: string): Promise<string>;
  deleteSecret(name: string): Promise<void>;
}

class AWSSecretsManager implements SecretProvider {
  async getSecret(name: string): Promise<string> {
    const secret = await secretsManager.getSecretValue({
      SecretId: name,
    }).promise();
    
    return secret.SecretString;
  }
  
  async setSecret(name: string, value: string): Promise<void> {
    await secretsManager.putSecretValue({
      SecretId: name,
      SecretString: value,
    }).promise();
  }
  
  async rotateSecret(name: string): Promise<string> {
    const rotationLambda = process.env.SECRET_ROTATION_LAMBDA;
    await lambda.invoke({
      FunctionName: rotationLambda,
      InvocationType: 'RequestResponse',
      Payload: JSON.stringify({ secretName: name }),
    }).promise();
    
    return this.getSecret(name);
  }
}
```

---

## Monitoring and Alerting

### Health Checks

```typescript
interface HealthCheckConfig {
  path: string;
  interval: number;
  timeout: number;
  unhealthyThreshold: number;
  healthyThreshold: number;
}

async function setupHealthChecks(
  serviceId: string,
  config: HealthCheckConfig
): Promise<void> {
  await loadBalancer.configureHealthCheck({
    Target: `HTTP:${config.path}`,
    Interval: config.interval,
    Timeout: config.timeout,
    UnhealthyThreshold: config.unhealthyThreshold,
    HealthyThreshold: config.healthyThreshold,
  });
  
  await serviceDiscovery.registerHealthCheck(
    serviceId,
    `http://${serviceId}:${process.env.PORT}${config.path}`
  );
}

async function checkApplicationHealth(): Promise<HealthStatus> {
  const checks = await Promise.all([
    checkDatabaseHealth(),
    checkCacheHealth(),
    checkExternalServiceHealth(),
    checkMemoryUsage(),
    checkErrorRate(),
  ]);
  
  const overallStatus = checks.every(c => c.healthy)
    ? 'healthy'
    : checks.some(c => c.critical)
      ? 'unhealthy'
      : 'degraded';
  
  return {
    status: overallStatus,
    timestamp: new Date().toISOString(),
    checks,
    uptime: process.uptime(),
    memoryUsage: process.memoryUsage(),
  };
}
```

### Alerting Configuration

```typescript
interface AlertRule {
  name: string;
  condition: AlertCondition;
  severity: 'critical' | 'warning' | 'info';
  notificationChannels: string[];
  runbook?: string;
}

const alertRules: AlertRule[] = [
  {
    name: 'HighErrorRate',
    condition: {
      metric: 'error_rate',
      operator: '>',
      threshold: 0.05,
      duration: '5m',
    },
    severity: 'critical',
    notificationChannels: ['pagerduty', 'slack-critical'],
    runbook: '/runbooks/high-error-rate.md',
  },
  {
    name: 'HighLatency',
    condition: {
      metric: 'p99_latency_ms',
      operator: '>',
      threshold: 1000,
      duration: '5m',
    },
    severity: 'warning',
    notificationChannels: ['slack-ops'],
    runbook: '/runbooks/high-latency.md',
  },
  {
    name: 'LowDiskSpace',
    condition: {
      metric: 'disk_free_percent',
      operator: '<',
      threshold: 15,
      duration: '15m',
    },
    severity: 'warning',
    notificationChannels: ['slack-ops', 'pagerduty'],
    runbook: '/runbooks/disk-space.md',
  },
];

async function configureAlerts(rules: AlertRule[]): Promise<void> {
  for (const rule of rules) {
    await monitoringClient.createAlertRule({
      Name: rule.name,
      MetricName: rule.condition.metric,
      ComparisonOperator: rule.condition.operator,
      Threshold: rule.condition.threshold,
      EvaluationPeriods: parseDuration(rule.condition.duration),
      TreatMissingData: 'notBreaching',
      AlarmActions: rule.notificationChannels.map(channel => 
        getNotificationArn(channel)
      ),
      OKActions: rule.notificationChannels,
    });
  }
}
```

---

## Rollback Procedures

### Rollback Strategy

```typescript
interface RollbackConfig {
  automaticTriggers: RollbackTrigger[];
  manualApprovalRequired: boolean;
  rollbackVersions: string[];
  dataRecovery: boolean;
}

interface RollbackTrigger {
  condition: string;
  metrics: string[];
  threshold: number;
  durationSeconds: number;
}

async function executeRollback(
  deploymentId: string,
  reason: string
): Promise<RollbackResult> {
  const deployment = await getDeployment(deploymentId);
  
  const targetVersion = deployment.previousVersion || 
    await findLastStableVersion();
  
  console.log(`Rolling back from ${deployment.version} to ${targetVersion}`);
  
  const rollbackTask = await createRollbackTask({
    fromVersion: deployment.version,
    toVersion: targetVersion,
    reason,
    initiatedBy: getCurrentUser(),
    timestamp: new Date(),
  });
  
  if (deployment.strategy === 'blue-green') {
    await executeBlueGreenRollback(deployment, targetVersion);
  } else if (deployment.strategy === 'canary') {
    await executeCanaryRollback(deployment);
  } else if (deployment.strategy === 'rolling') {
    await executeRollingRollback(deployment, targetVersion);
  }
  
  const verification = await verifyRollback(targetVersion);
  
  await finalizeRollback(rollbackTask.id, verification.success);
  
  await sendRollbackNotification({
    deploymentId,
    fromVersion: deployment.version,
    toVersion: targetVersion,
    reason,
    success: verification.success,
  });
  
  return {
    success: verification.success,
    fromVersion: deployment.version,
    toVersion: targetVersion,
    verification,
  };
}
```

### Rollback Automation

```yaml
name: Rollback

on:
  workflow_dispatch:
    inputs:
      deployment_id:
        description: 'Deployment ID to rollback'
        required: true
      reason:
        description: 'Reason for rollback'
        required: true
      target_version:
        description: 'Version to rollback to (optional)'
        required: false

jobs:
  rollback:
    name: Execute Rollback
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}
      
      - name: Execute rollback
        run: |
          if [ -n "${{ github.inputs.target_version }}" ]; then
            TARGET_VERSION="${{ github.inputs.target_version }}"
          else
            TARGET_VERSION=$(echo '${{ steps.deployment.outputs.deployment }}' | \
              jq -r '.[] | select(.status=="PRIMARY") | .taskDefinition' | \
              grep -oP ':\K[0-9]+$')
          fi
          aws ecs update-service \
            --cluster ${{ secrets.ECS_CLUSTER }} \
            --service ${{ secrets.ECS_SERVICE }} \
            --task-definition ${{ secrets.TASK_FAMILY }}:$TARGET_VERSION \
            --force-new-deployment
      
      - name: Verify rollback
        run: |
          sleep 30
          aws ecs wait services-stable \
            --cluster ${{ secrets.ECS_CLUSTER }} \
            --services ${{ secrets.ECS_SERVICE }}
          
          HEALTH=$(curl -f ${{ secrets.HEALTH_ENDPOINT }}/health || echo "FAILED")
          if [ "$HEALTH" != "FAILED" ]; then
            echo "Rollback verification passed"
          else
            echo "Rollback verification failed"
            exit 1
          }
```

---

## Git Workflow and Version Control

### Conventional Commits

```
<type>[optional scope]: <description>

[optional body]

[optional footer]
```

#### Commit Types

| Type | Description | Examples |
|------|-------------|----------|
| `feat` | New feature | `feat: add user registration` |
| `fix` | Bug fix | `fix: handle null email validation` |
| `docs` | Documentation only | `docs: update README installation` |
| `style` | Code style changes | `style: format with prettier` |
| `refactor` | Code refactoring | `refactor: extract validation logic` |
| `perf` | Performance improvements | `perf: optimize query caching` |
| `test` | Adding or updating tests | `test: add integration tests for auth` |
| `build` | Build system changes | `build: update webpack config` |
| `ci` | CI/CD changes | `ci: add GitHub Actions workflow` |
| `chore` | Other changes | `chore: update dependencies` |
| `revert` | Revert previous commit | `revert: feat(user): remove deprecated field` |

### Branch Naming

#### Feature Branches

```
feat/<feature-name>
feat/user-registration
feat/jwt-auth
```

#### Bugfix Branches

```
fix/<bug-description>
fix/email-validation-error
fix/database-timeout
```

#### Hotfix Branches

```
hotfix/<hotfix-description>
hotfix/security-patch-2024-01
hotfix/critical-bug-123
```

#### Release Branches

```
release/<version>
release/v1.0.0
release/v1.1.0
```

### Version Tags

```bash
# Tag release
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# Format: v<MAJOR>.<MINOR>.<PATCH>
# v1.0.0 - Initial release
# v1.1.0 - New feature (backward compatible)
# v1.1.1 - Bug fix (backward compatible)
# v2.0.0 - Breaking changes
```

### Commitlint Configuration

```javascript
// commitlint.config.js
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'build', 'ci', 'chore', 'revert'],
    ],
    'subject-case': [0],
    'subject-max-length': [2, 'always', 72],
    'body-max-line-length': [2, 'always', 100],
  },
};
```

---

## AI-Assisted Deployment Guidelines

When working with CI/CD pipelines and deployment as an AI assistant:

1. **Understand the Pipeline Flow**: Read through the entire pipeline configuration before making changes. Understand the dependencies between stages.

2. **Preserve Artifact Pass-Through**: Ensure build artifacts are properly passed between jobs using artifact storage mechanisms.

3. **Environment-Specific Configuration**: Use environment variables and secrets appropriately for different deployment targets.

4. **Fail Fast**: Configure quality gates to fail early in the pipeline to save resources and time.

5. **Include Rollback Strategy**: Always consider how to rollback if a deployment fails.

6. **Security Scanning**: Recommend security scanning at appropriate stages without slowing down the pipeline unnecessarily.

7. **Caching Strategy**: Suggest caching for dependencies, build outputs, and Docker layers to speed up pipeline execution.

8. **Understand Environment Differences**: Staging should mirror production but with appropriate data isolation and security controls.

9. **Data Privacy First**: Never suggest copying production data to staging without proper anonymization and compliance checks.

10. **Deployment Safety**: Always include rollback procedures and health checks in deployment strategies.

11. **Infrastructure as Code**: Use Terraform/CloudFormation/Pulumi for environment configuration to ensure reproducibility.

12. **Secret Management**: Recommend using secret management services (AWS Secrets Manager, HashiCorp Vault) rather than environment variables for sensitive data.

13. **Monitoring First**: Suggest setting up monitoring and alerting before deploying to any environment.

14. **Document Everything**: Ensure runbooks, deployment procedures, and rollback steps are documented and accessible.

15. **Test Rollbacks**: Regularly practice rollback procedures to ensure they work when needed.

---

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

### Anti-Pattern: Identical Staging and Production

```typescript
async function badStagingSetup(): Promise<void> {
  const productionData = await copyProductionDatabase('staging');
  await restoreDatabase(productionData, 'staging';
}
```

### Anti-Pattern: Manual Deployments

```typescript
async function badManualDeployment(): Promise<void> {
  await ssh.connect('production-server', { user: 'admin' });
  await ssh.execCommand('cd /var/www/app && git pull');
  await ssh.execCommand('npm install --production');
  await ssh.execCommand('pm2 restart all');
}
```

### Better: Automated Deployment Pipeline

```typescript
async function goodAutomatedDeployment(): Promise<void> {
  const deployment = await createDeployment({
    version: getVersionFromTag(),
    environment: 'production',
    strategy: 'blue-green',
    requiresApproval: true,
  });
  
  await executeDeployment(deployment);
  
  const verification = await verifyDeployment({
    healthCheckPath: '/health',
    smokeTestPath: '/api/smoke',
    metricsCheck: ['error_rate', 'latency_p99'],
  });
  
  if (!verification.passed) {
    await executeRollback(deployment.id, 'Verification failed');
  }
}
```

---

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)
- [AWS Deployment Strategies](https://aws.amazon.com/devops/continuous-delivery/)
- [12-Factor App: Config](https://12factor.net/config)
