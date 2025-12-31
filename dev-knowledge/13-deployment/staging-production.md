# Staging and Production Environments

Proper environment management is essential for reliable software delivery. Staging and production environments serve distinct purposes in the deployment lifecycle.

## Environment Overview

### Environment Comparison

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

## Staging Environment

### Purpose and Goals

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

### Staging Setup

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

### Data Management

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

## Production Environment

### Production Readiness

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

## Environment Configuration

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

### Secret Management

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

## Common Anti-Patterns

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

## AI Guidelines

When working with staging and production environments as an AI assistant:

1. **Understand Environment Differences**: Staging should mirror production but with appropriate data isolation and security controls.

2. **Data Privacy First**: Never suggest copying production data to staging without proper anonymization and compliance checks.

3. **Deployment Safety**: Always include rollback procedures and health checks in deployment strategies.

4. **Infrastructure as Code**: Use Terraform/CloudFormation/Pulumi for environment configuration to ensure reproducibility.

5. **Secret Management**: Recommend using secret management services (AWS Secrets Manager, HashiCorp Vault) rather than environment variables for sensitive data.

6. **Monitoring First**: Suggest setting up monitoring and alerting before deploying to any environment.

7. **Document Everything**: Ensure runbooks, deployment procedures, and rollback steps are documented and accessible.

8. **Test Rollbacks**: Regularly practice rollback procedures to ensure they work when needed.
