# Complete Jenkins & CI/CD Interview Guide

Master Jenkins with 12 real-world interview questions covering pipeline debugging, shared libraries, distributed builds, security, and production operations. Practice scenarios that mirror actual senior DevOps/SRE engineer challenges.

**Companies that ask these questions:** Netflix | LinkedIn | Amazon | Microsoft | Uber | Spotify

## Questions Overview

| # | Question | Type | Category |
|---|----------|------|----------|
| 1 | A critical deployment pipeline is failing intermittently in ... | Debugging | Pipeline Debugging |
| 2 | 50 teams are duplicating pipeline code. Design a shared libr... | Architecture | Shared Libraries |
| 3 | Builds are taking 45 minutes and blocking deployments. Optim... | Practical | Build Optimization |
| 4 | Credentials are hardcoded in pipelines and logs expose secre... | Practical | Security & Credentials |
| 5 | Single Jenkins server is overloaded. Design a distributed bu... | Architecture | Distributed Builds |
| 6 | Teams need automated CI for all branches with different depl... | Practical | Multi-branch Pipelines |
| 7 | Jenkins agents keep disconnecting and builds are stuck in qu... | Debugging | Agent Management |
| 8 | Migrate from static VMs to Kubernetes-based dynamic agents f... | Practical | Kubernetes Integration |
| 9 | Everyone has admin access to Jenkins. Implement proper RBAC ... | Practical | Security & RBAC |
| 10 | Jenkins configuration is manual and inconsistent. Implement ... | Practical | Configuration as Code |
| 11 | Design a GitOps workflow with Jenkins for automated, auditab... | Architecture | GitOps & Deployment |
| 12 | Jenkins server crashed and we lost all job configurations. I... | Practical | Backup & Recovery |

---

## What You'll Learn

- Debug failing pipelines with proper error handling and retry logic
- Design reusable shared libraries for enterprise-scale Jenkins
- Optimize build times with parallel execution and caching
- Implement secure credential management with HashiCorp Vault integration
- Configure distributed builds with dynamic agent provisioning
- Design multi-branch pipelines with proper environment promotion
- Implement GitOps workflows with automated deployments
- Secure Jenkins with RBAC, audit logging, and hardening
- Integrate Jenkins with Kubernetes for cloud-native CI/CD
- Handle disaster recovery with proper backup strategies

---

## Interview Questions

### Question 1: A critical deployment pipeline is failing intermittently in production. Debug and fix the issue.

**Type:** Debugging | **Category:** Pipeline Debugging

## The Scenario

It's Friday afternoon and the release train is blocked. Your deployment pipeline has been failing intermittently for the past hour:

```
Started by user deploy-bot
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on agent-prod-01 in /var/jenkins/workspace/deploy-production
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Deploy to Production)
[Pipeline] sh
+ kubectl apply -f k8s/deployment.yaml
error: unable to connect to the server: dial tcp 10.0.1.50:6443: i/o timeout
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
ERROR: script returned exit code 1
Finished: FAILURE
```

The same pipeline worked 3 hours ago. Nothing in the Jenkinsfile changed. The Kubernetes cluster is healthy when you check manually.

## The Challenge

Debug why the pipeline fails intermittently, identify the root cause, and implement a robust fix with proper error handling and retry logic.


### Step 1: Check Agent and Network Status

```groovy
// Add debugging to your pipeline
pipeline {
    agent { label 'prod-agents' }

    environment {
        KUBECONFIG = credentials('kubeconfig-prod')
    }

    stages {
        stage('Debug Connectivity') {
            steps {
                script {
                    // Check agent details
                    sh '''
                        echo "=== Agent Information ==="
                        hostname
                        ip addr show
                        cat /etc/resolv.conf

                        echo "=== Kubernetes API Connectivity ==="
                        curl -v -k https://10.0.1.50:6443/healthz --max-time 10 || true

                        echo "=== DNS Resolution ==="
                        nslookup kubernetes.default.svc.cluster.local || true

                        echo "=== Route to API Server ==="
                        traceroute -m 5 10.0.1.50 || true
                    '''
                }
            }
        }
    }
}
```

### Step 2: Identify Common Root Causes

```groovy
// Check for credential expiration
stage('Validate Credentials') {
    steps {
        script {
            // Test kubeconfig validity
            def result = sh(
                script: '''
                    kubectl cluster-info --request-timeout=10s 2>&1
                ''',
                returnStatus: true
            )

            if (result != 0) {
                // Check if token expired
                sh '''
                    echo "Checking token expiration..."
                    kubectl config view --raw -o jsonpath='{.users[0].user.token}' | \
                        cut -d'.' -f2 | base64 -d 2>/dev/null | jq -r '.exp' | \
                        xargs -I {} date -d @{} || echo "Token check failed"
                '''
                error("Kubernetes credentials may be expired or invalid")
            }
        }
    }
}
```

### Step 3: Implement Robust Retry Logic

```groovy
pipeline {
    agent { label 'prod-agents' }

    options {
        timeout(time: 30, unit: 'MINUTES')
        retry(2)  // Retry entire pipeline on failure
    }

    environment {
        KUBECONFIG = credentials('kubeconfig-prod')
    }

    stages {
        stage('Pre-flight Checks') {
            steps {
                script {
                    // Verify connectivity before proceeding
                    retry(3) {
                        sh '''
                            kubectl cluster-info --request-timeout=15s
                        '''
                    }
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    def maxRetries = 3
                    def retryDelay = 10

                    for (int i = 0; i < maxRetries; i++) {
                        try {
                            sh '''
                                kubectl apply -f k8s/deployment.yaml --timeout=60s
                                kubectl rollout status deployment/app --timeout=300s
                            '''
                            echo "Deployment successful on attempt ${i + 1}"
                            break
                        } catch (Exception e) {
                            if (i == maxRetries - 1) {
                                error("Deployment failed after ${maxRetries} attempts: ${e.message}")
                            }
                            echo "Attempt ${i + 1} failed, retrying in ${retryDelay} seconds..."
                            sleep(retryDelay)
                            retryDelay *= 2  // Exponential backoff
                        }
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                // Collect diagnostic information on failure
                sh '''
                    echo "=== Collecting diagnostics ==="
                    kubectl get nodes -o wide || true
                    kubectl get pods -n kube-system || true
                    kubectl describe nodes | grep -A 5 "Conditions:" || true
                '''
            }
        }
    }
}
```

### Step 4: Add Health Checks and Circuit Breaker

```groovy
def checkKubernetesHealth() {
    def healthChecks = [
        'API Server': 'kubectl cluster-info --request-timeout=10s',
        'Node Status': 'kubectl get nodes --request-timeout=10s | grep -v NotReady',
        'Core DNS': 'kubectl get pods -n kube-system -l k8s-app=kube-dns --request-timeout=10s'
    ]

    def failures = []

    healthChecks.each { name, command ->
        def result = sh(script: command, returnStatus: true)
        if (result != 0) {
            failures.add(name)
        }
    }

    if (failures.size() > 0) {
        error("Health checks failed: ${failures.join(', ')}")
    }

    echo "All health checks passed"
}

// Use in pipeline
stage('Health Check') {
    steps {
        script {
            checkKubernetesHealth()
        }
    }
}
```

### Step 5: Implement Proper Error Handling

```groovy
pipeline {
    agent { label 'prod-agents' }

    stages {
        stage('Deploy') {
            steps {
                script {
                    try {
                        timeout(time: 5, unit: 'MINUTES') {
                            sh 'kubectl apply -f k8s/deployment.yaml'
                        }
                    } catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException e) {
                        echo "Deployment timed out - checking cluster status"
                        sh 'kubectl get events --sort-by=.lastTimestamp | tail -20'
                        throw e
                    } catch (hudson.AbortException e) {
                        echo "kubectl command failed - checking connectivity"
                        sh 'curl -k https://kubernetes.default.svc/healthz || true'
                        throw e
                    }
                }
            }
        }
    }
}
```


## Common Pipeline Debugging Issues

| Symptom | Root Cause | Fix |
|---------|------------|-----|
| Intermittent timeout | Network instability or agent overload | Add retry with backoff, check agent resources |
| Connection refused | API server overloaded or firewall | Check server health, verify security groups |
| Certificate errors | Expired or mismatched certs | Refresh credentials, check cert validity |
| Permission denied | RBAC changes or token expiration | Verify service account, refresh tokens |
| Resource not found | Wrong namespace or context | Verify KUBECONFIG context and namespace |

## Debugging Commands Cheat Sheet

```bash
# Check Jenkins agent logs
tail -f /var/log/jenkins/jenkins.log

# Verify agent connectivity
curl -I http://jenkins-master:8080/computer/agent-name/

# Test Kubernetes from agent
kubectl auth can-i --list

# Check for network issues
netstat -tulpn | grep 6443
ss -tulpn | grep ESTAB
```

---

### Question 2: 50 teams are duplicating pipeline code. Design a shared library architecture for the organization.

**Type:** Architecture | **Category:** Shared Libraries

## The Scenario

Your organization has grown to 50 engineering teams, each with their own Jenkins pipelines. You notice:

- Every team has copy-pasted the same deployment code with slight variations
- Security vulnerabilities found in one pipeline exist in 200+ others
- Teams spend days writing boilerplate instead of shipping features
- No standardization for notifications, approvals, or rollback procedures

Leadership wants a solution that enables consistency while allowing team flexibility.

## The Challenge

Design a shared library architecture that provides reusable, versioned pipeline components while allowing teams to customize behavior for their specific needs.


> **Common Mistake:** A junior engineer might create one giant shared library with all logic, force all teams to use identical pipelines, or just document best practices and hope teams follow them. These approaches fail because monolithic libraries are hard to maintain, forced standardization kills productivity, and documentation without enforcement leads to drift.

> **Senior Engineer Approach:** A senior engineer designs a modular shared library with clear separation of concerns, versioned releases for stability, sensible defaults with override capabilities, and comprehensive testing. They implement a governance model that balances standardization with team autonomy.

### Step 1: Design Library Structure

```
jenkins-shared-library/
├── vars/                          # Global variables (pipeline steps)
│   ├── standardPipeline.groovy    # Full pipeline template
│   ├── buildApp.groovy            # Build step
│   ├── deployToK8s.groovy         # Kubernetes deployment
│   ├── notifySlack.groovy         # Slack notifications
│   ├── securityScan.groovy        # Security scanning
│   └── runTests.groovy            # Test execution
├── src/
│   └── org/
│       └── company/
│           └── jenkins/
│               ├── Config.groovy          # Configuration classes
│               ├── DeploymentStrategy.groovy
│               ├── NotificationService.groovy
│               └── SecurityValidator.groovy
├── resources/
│   ├── templates/
│   │   ├── kubernetes/
│   │   │   ├── deployment.yaml
│   │   │   └── service.yaml
│   │   └── docker/
│   │       └── Dockerfile.template
│   └── scripts/
│       └── security-scan.sh
└── test/
    └── groovy/
        └── vars/
            └── StandardPipelineTest.groovy
```

### Step 2: Create Core Pipeline Template

```groovy
// vars/standardPipeline.groovy
def call(Map config = [:]) {
    // Validate required parameters
    validateConfig(config)

    // Merge with defaults
    def pipelineConfig = mergeWithDefaults(config)

    pipeline {
        agent { label pipelineConfig.agentLabel ?: 'default' }

        options {
            buildDiscarder(logRotator(numToKeepStr: '10'))
            timeout(time: pipelineConfig.timeoutMinutes ?: 60, unit: 'MINUTES')
            timestamps()
            ansiColor('xterm')
        }

        environment {
            APP_NAME = pipelineConfig.appName
            TEAM = pipelineConfig.team
        }

        stages {
            stage('Checkout') {
                steps {
                    checkout scm
                }
            }

            stage('Build') {
                when { expression { pipelineConfig.enableBuild != false } }
                steps {
                    script {
                        buildApp(pipelineConfig.build ?: [:])
                    }
                }
            }

            stage('Test') {
                when { expression { pipelineConfig.enableTests != false } }
                parallel {
                    stage('Unit Tests') {
                        steps {
                            runTests(type: 'unit', config: pipelineConfig.tests?.unit ?: [:])
                        }
                    }
                    stage('Integration Tests') {
                        when { expression { pipelineConfig.tests?.integration?.enabled != false } }
                        steps {
                            runTests(type: 'integration', config: pipelineConfig.tests?.integration ?: [:])
                        }
                    }
                }
            }

            stage('Security Scan') {
                when { expression { pipelineConfig.enableSecurityScan != false } }
                steps {
                    securityScan(pipelineConfig.security ?: [:])
                }
            }

            stage('Deploy to Dev') {
                when {
                    branch 'develop'
                    expression { pipelineConfig.environments?.dev?.enabled != false }
                }
                steps {
                    deployToK8s(
                        environment: 'dev',
                        config: pipelineConfig.environments?.dev ?: [:]
                    )
                }
            }

            stage('Deploy to Production') {
                when {
                    branch 'main'
                    expression { pipelineConfig.environments?.prod?.enabled != false }
                }
                steps {
                    script {
                        // Require approval for production
                        if (pipelineConfig.environments?.prod?.requireApproval != false) {
                            approveDeployment(
                                environment: 'production',
                                approvers: pipelineConfig.environments?.prod?.approvers ?: ['platform-team']
                            )
                        }

                        deployToK8s(
                            environment: 'prod',
                            config: pipelineConfig.environments?.prod ?: [:]
                        )
                    }
                }
            }
        }

        post {
            always {
                script {
                    notifySlack(
                        channel: pipelineConfig.notifications?.slackChannel ?: '#builds',
                        status: currentBuild.result
                    )
                }
            }
            failure {
                script {
                    if (pipelineConfig.notifications?.pagerduty?.enabled) {
                        triggerPagerDuty(pipelineConfig.notifications.pagerduty)
                    }
                }
            }
        }
    }
}

def validateConfig(Map config) {
    def required = ['appName', 'team']
    def missing = required.findAll { !config.containsKey(it) }
    if (missing) {
        error "Missing required configuration: ${missing.join(', ')}"
    }
}

def mergeWithDefaults(Map config) {
    def defaults = [
        agentLabel: 'docker',
        timeoutMinutes: 60,
        enableBuild: true,
        enableTests: true,
        enableSecurityScan: true
    ]
    return defaults + config
}
```

### Step 3: Create Modular Components

```groovy
// vars/buildApp.groovy
def call(Map config = [:]) {
    def buildType = config.type ?: detectBuildType()

    switch(buildType) {
        case 'maven':
            buildMaven(config)
            break
        case 'gradle':
            buildGradle(config)
            break
        case 'npm':
            buildNpm(config)
            break
        case 'docker':
            buildDocker(config)
            break
        default:
            error "Unknown build type: ${buildType}"
    }
}

def buildDocker(Map config) {
    def imageName = config.imageName ?: "${env.APP_NAME}"
    def registry = config.registry ?: 'registry.company.com'
    def tag = config.tag ?: "${env.GIT_COMMIT?.take(8) ?: 'latest'}"

    withCredentials([usernamePassword(
        credentialsId: config.registryCredentials ?: 'docker-registry',
        usernameVariable: 'DOCKER_USER',
        passwordVariable: 'DOCKER_PASS'
    )]) {
        sh """
            echo \$DOCKER_PASS | docker login ${registry} -u \$DOCKER_USER --password-stdin
            docker build -t ${registry}/${imageName}:${tag} .
            docker push ${registry}/${imageName}:${tag}
        """
    }

    // Return image reference for downstream stages
    return "${registry}/${imageName}:${tag}"
}

def detectBuildType() {
    if (fileExists('pom.xml')) return 'maven'
    if (fileExists('build.gradle')) return 'gradle'
    if (fileExists('package.json')) return 'npm'
    if (fileExists('Dockerfile')) return 'docker'
    error "Could not detect build type. Please specify config.type"
}
```

```groovy
// vars/deployToK8s.groovy
def call(Map args) {
    def environment = args.environment
    def config = args.config ?: [:]

    def namespace = config.namespace ?: "${env.APP_NAME}-${environment}"
    def kubeconfig = config.kubeconfig ?: "kubeconfig-${environment}"

    withCredentials([file(credentialsId: kubeconfig, variable: 'KUBECONFIG')]) {
        script {
            // Apply Kubernetes manifests
            def manifests = config.manifests ?: "k8s/${environment}"

            // Substitute environment variables
            sh """
                envsubst < ${manifests}/deployment.yaml | kubectl apply -f -
                envsubst < ${manifests}/service.yaml | kubectl apply -f -
            """

            // Wait for rollout
            def deploymentName = config.deploymentName ?: env.APP_NAME
            timeout(time: config.rolloutTimeout ?: 10, unit: 'MINUTES') {
                sh """
                    kubectl rollout status deployment/${deploymentName} \
                        -n ${namespace} \
                        --timeout=600s
                """
            }

            // Run smoke tests if configured
            if (config.smokeTest) {
                runSmokeTest(config.smokeTest)
            }
        }
    }
}
```

### Step 4: Enable Team Customization

```groovy
// Team's Jenkinsfile - Simple usage with defaults
@Library('company-shared-library@v2.0') _

standardPipeline(
    appName: 'user-service',
    team: 'platform'
)
```

```groovy
// Team's Jenkinsfile - Full customization
@Library('company-shared-library@v2.0') _

standardPipeline(
    appName: 'payment-service',
    team: 'payments',
    agentLabel: 'secure-agents',
    timeoutMinutes: 90,

    build: [
        type: 'docker',
        imageName: 'payment-service',
        registry: 'registry.company.com/payments'
    ],

    tests: [
        unit: [enabled: true],
        integration: [
            enabled: true,
            database: 'postgres:13'
        ]
    ],

    security: [
        scanners: ['snyk', 'trivy'],
        failOnCritical: true
    ],

    environments: [
        dev: [enabled: true, namespace: 'payments-dev'],
        staging: [enabled: true, namespace: 'payments-staging'],
        prod: [
            enabled: true,
            namespace: 'payments-prod',
            requireApproval: true,
            approvers: ['payments-leads', 'sre-team']
        ]
    ],

    notifications: [
        slackChannel: '#payments-builds',
        pagerduty: [enabled: true, serviceKey: 'payments-pd']
    ]
)
```

### Step 5: Versioning and Release Strategy

```groovy
// Jenkinsfile for the shared library itself
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh './gradlew test'
            }
        }

        stage('Release') {
            when { branch 'main' }
            steps {
                script {
                    def version = sh(
                        script: 'git describe --tags --abbrev=0',
                        returnStdout: true
                    ).trim()

                    // Tag and push
                    sh """
                        git tag -a v${version} -m "Release ${version}"
                        git push origin v${version}
                    """
                }
            }
        }
    }
}
```

Configure in Jenkins:
```
Manage Jenkins > Configure System > Global Pipeline Libraries

Name: company-shared-library
Default version: main
Allow default version to be overridden: true
Include @Library changes in job recent changes: true

Retrieval method: Modern SCM
  - Git
  - Project Repository: https://github.com/company/jenkins-shared-library
  - Behaviors: Discover branches, Discover tags
```


## Shared Library Best Practices

| Practice | Why | How |
|----------|-----|-----|
| Semantic versioning | Teams can pin to stable versions | Use git tags: v1.0.0, v2.0.0 |
| Comprehensive testing | Catch bugs before teams hit them | Unit test with JenkinsPipelineUnit |
| Sensible defaults | Teams shouldn't need to configure everything | Provide working defaults, allow overrides |
| Clear documentation | Teams need to know what's available | README, examples, changelog |
| Deprecation policy | Allow time for teams to migrate | Warn for 2 versions before removing |

<InterviewQuiz
  question="What is the primary advantage of using vars/ directory over src/ directory for shared library functions in Jenkins?"
  options={[
    "vars/ provides better performance",
    "vars/ functions are automatically available as pipeline steps without imports",
    "src/ doesn't support Groovy classes",
    "vars/ has better security"
  ]}
  correctAnswer={1}
  explanation="Files in the vars/ directory are automatically exposed as global pipeline steps. A file named 'deployToK8s.groovy' with a 'call()' method becomes available as deployToK8s() in any pipeline that loads the library. The src/ directory requires explicit imports and is better suited for supporting classes and utilities that don't need to be directly called as pipeline steps."
/>

---

### Question 3: Builds are taking 45 minutes and blocking deployments. Optimize to under 10 minutes.

**Type:** Practical | **Category:** Build Optimization

## The Scenario

Your team's Jenkins pipeline takes 45 minutes to complete:

```
[Pipeline] Stage: Checkout         - 2 min
[Pipeline] Stage: Install Deps     - 8 min
[Pipeline] Stage: Build            - 12 min
[Pipeline] Stage: Unit Tests       - 10 min
[Pipeline] Stage: Integration Tests - 8 min
[Pipeline] Stage: Security Scan    - 3 min
[Pipeline] Stage: Docker Build     - 5 min
[Pipeline] Stage: Push & Deploy    - 2 min
Total: ~45 minutes
```

Developers are frustrated. They push code and wait nearly an hour for feedback. Deployment frequency has dropped because the pipeline is a bottleneck.

## The Challenge

Optimize the pipeline to complete in under 10 minutes while maintaining test coverage and security scanning.


> **Common Mistake:** A junior engineer might skip tests to save time, throw more hardware at the problem, or remove security scans entirely. These approaches create technical debt, waste money, and introduce security vulnerabilities that cost far more than the time saved.

> **Senior Engineer Approach:** A senior engineer analyzes each stage to identify parallelization opportunities, implements caching at every layer, optimizes resource-intensive operations, and restructures the pipeline for maximum efficiency without sacrificing quality.

### Step 1: Analyze Current Pipeline

```groovy
// Add timing to identify bottlenecks
def timings = [:]

def timedStage(String name, Closure body) {
    def start = System.currentTimeMillis()
    stage(name) {
        body()
    }
    def duration = System.currentTimeMillis() - start
    timings[name] = duration
    echo "Stage '${name}' took ${duration/1000}s"
}

pipeline {
    agent any

    stages {
        stage('Timed Build') {
            steps {
                script {
                    timedStage('Checkout') { checkout scm }
                    timedStage('Install') { sh 'npm ci' }
                    timedStage('Build') { sh 'npm run build' }
                    timedStage('Test') { sh 'npm test' }
                }
            }
        }
    }

    post {
        always {
            script {
                echo "=== Build Timing Summary ==="
                timings.each { stage, ms ->
                    echo "${stage}: ${ms/1000}s"
                }
            }
        }
    }
}
```

### Step 2: Parallelize Independent Stages

```groovy
pipeline {
    agent { label 'docker' }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                stash includes: '**', name: 'source'
            }
        }

        stage('Parallel Build & Test') {
            parallel {
                stage('Build') {
                    agent { label 'docker' }
                    steps {
                        unstash 'source'
                        sh 'npm ci --prefer-offline'
                        sh 'npm run build'
                        stash includes: 'dist/**', name: 'build'
                    }
                }

                stage('Unit Tests') {
                    agent { label 'docker' }
                    steps {
                        unstash 'source'
                        sh 'npm ci --prefer-offline'
                        sh 'npm run test:unit -- --maxWorkers=4'
                    }
                }

                stage('Lint & Type Check') {
                    agent { label 'docker' }
                    steps {
                        unstash 'source'
                        sh 'npm ci --prefer-offline'
                        sh 'npm run lint & npm run typecheck & wait'
                    }
                }

                stage('Security Scan') {
                    agent { label 'docker' }
                    steps {
                        unstash 'source'
                        sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
                    }
                }
            }
        }

        stage('Integration Tests') {
            agent { label 'docker' }
            steps {
                unstash 'source'
                sh 'npm ci --prefer-offline'
                sh 'npm run test:integration'
            }
        }

        stage('Docker Build & Push') {
            agent { label 'docker' }
            steps {
                unstash 'build'
                sh '''
                    docker build -t app:${GIT_COMMIT} .
                    docker push registry.company.com/app:${GIT_COMMIT}
                '''
            }
        }
    }
}
```

### Step 3: Implement Aggressive Caching

```groovy
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: node
    image: node:18
    command: ['sleep', 'infinity']
    volumeMounts:
    - name: npm-cache
      mountPath: /root/.npm
    - name: node-modules-cache
      mountPath: /home/jenkins/workspace/node_modules_cache
  volumes:
  - name: npm-cache
    persistentVolumeClaim:
      claimName: npm-cache-pvc
  - name: node-modules-cache
    persistentVolumeClaim:
      claimName: node-modules-cache-pvc
'''
        }
    }

    stages {
        stage('Install with Cache') {
            steps {
                container('node') {
                    script {
                        // Check if node_modules cache exists and matches package-lock.json
                        def lockHash = sh(
                            script: 'md5sum package-lock.json | cut -d" " -f1',
                            returnStdout: true
                        ).trim()

                        def cacheDir = "/home/jenkins/workspace/node_modules_cache/${lockHash}"

                        if (fileExists(cacheDir)) {
                            sh "cp -r ${cacheDir}/node_modules ."
                            echo "Restored node_modules from cache"
                        } else {
                            sh 'npm ci'
                            sh "mkdir -p ${cacheDir} && cp -r node_modules ${cacheDir}/"
                            echo "Cached node_modules for future builds"
                        }
                    }
                }
            }
        }
    }
}
```

### Step 4: Optimize Docker Builds

```dockerfile
# Optimized Dockerfile with multi-stage build and layer caching
# Stage 1: Dependencies (cached unless package.json changes)
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Build (cached unless source changes)
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Production image (minimal)
FROM node:18-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

USER node
CMD ["node", "dist/index.js"]
```

```groovy
// Use BuildKit for faster Docker builds
stage('Docker Build') {
    steps {
        sh '''
            export DOCKER_BUILDKIT=1
            docker build \
                --cache-from registry.company.com/app:latest \
                --build-arg BUILDKIT_INLINE_CACHE=1 \
                -t registry.company.com/app:${GIT_COMMIT} \
                -t registry.company.com/app:latest \
                .
        '''
    }
}
```

### Step 5: Optimize Test Execution

```groovy
stage('Smart Testing') {
    steps {
        script {
            // Get changed files
            def changedFiles = sh(
                script: 'git diff --name-only HEAD~1',
                returnStdout: true
            ).trim().split('\n')

            // Determine which tests to run
            def testScope = 'all'
            if (changedFiles.every { it.startsWith('docs/') }) {
                testScope = 'none'
                echo "Only docs changed, skipping tests"
            } else if (changedFiles.every { it.startsWith('src/components/') }) {
                testScope = 'unit'
                echo "Only components changed, running unit tests only"
            }

            if (testScope == 'all' || testScope == 'unit') {
                sh '''
                    # Run tests in parallel with optimal workers
                    npm run test:unit -- \
                        --maxWorkers=50% \
                        --bail \
                        --changedSince=HEAD~1
                '''
            }

            if (testScope == 'all') {
                sh 'npm run test:integration'
            }
        }
    }
}
```

### Step 6: Complete Optimized Pipeline

```groovy
pipeline {
    agent none  // Use agents per-stage for parallelization

    options {
        skipDefaultCheckout()
        parallelsAlwaysFailFast()
    }

    stages {
        stage('Checkout') {
            agent { label 'lightweight' }
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: env.GIT_BRANCH]],
                    extensions: [
                        [$class: 'CloneOption', depth: 1, shallow: true],
                        [$class: 'CleanBeforeCheckout']
                    ],
                    userRemoteConfigs: [[url: env.GIT_URL, credentialsId: 'git-creds']]
                ])
                stash includes: '**', name: 'source'
            }
        }

        stage('Parallel Execution') {
            parallel {
                stage('Build & Unit Tests') {
                    agent { label 'docker-large' }
                    steps {
                        unstash 'source'
                        sh '''
                            npm ci --prefer-offline --cache .npm
                            npm run build &
                            npm run test:unit -- --maxWorkers=4 &
                            wait
                        '''
                        stash includes: 'dist/**', name: 'build'
                    }
                }

                stage('Security & Lint') {
                    agent { label 'docker' }
                    steps {
                        unstash 'source'
                        sh '''
                            npm ci --prefer-offline --cache .npm
                            npm run lint &
                            trivy fs --exit-code 1 --severity CRITICAL . &
                            wait
                        '''
                    }
                }
            }
        }

        stage('Integration Tests') {
            agent { label 'docker-large' }
            steps {
                unstash 'source'
                sh '''
                    npm ci --prefer-offline --cache .npm
                    npm run test:integration -- --maxWorkers=2
                '''
            }
        }

        stage('Docker Build & Deploy') {
            agent { label 'docker' }
            steps {
                unstash 'build'
                sh '''
                    export DOCKER_BUILDKIT=1
                    docker build --cache-from app:latest -t app:${GIT_COMMIT} .
                    docker push registry.company.com/app:${GIT_COMMIT}
                    kubectl set image deployment/app app=registry.company.com/app:${GIT_COMMIT}
                '''
            }
        }
    }
}
```


## Optimization Results

| Stage | Before | After | Technique |
|-------|--------|-------|-----------|
| Checkout | 2 min | 15 sec | Shallow clone |
| Install Deps | 8 min | 30 sec | npm cache + PVC |
| Build | 12 min | 2 min | Parallel + cache |
| Unit Tests | 10 min | 2 min | Parallel workers |
| Integration | 8 min | 3 min | Selective execution |
| Security Scan | 3 min | 1 min | Run in parallel |
| Docker Build | 5 min | 1 min | BuildKit + layer cache |
| **Total** | **45 min** | **8 min** | Combined |


---

### Quick Check

**Which optimization technique typically provides the biggest improvement for npm-based build pipelines?**

   A. Using faster hardware
-> B. **Caching node_modules across builds**
   C. Running fewer tests
   D. Using a different CI tool

<details>
<summary>See Answer</summary>

Caching node_modules (or using a persistent npm cache) typically provides the biggest improvement because npm install/ci is often the slowest step. With proper caching, subsequent builds can skip downloading and installing thousands of packages. This can reduce a 5-10 minute install to under 30 seconds. While hardware helps, caching addresses the root cause - repeatedly downloading the same packages.

</details>

---

### Question 4: Credentials are hardcoded in pipelines and logs expose secrets. Implement secure credential management.

**Type:** Practical | **Category:** Security & Credentials

## The Scenario

During a security audit, you discover critical issues:

```groovy
// Found in production Jenkinsfile
pipeline {
    environment {
        DB_PASSWORD = 'prod_p@ssw0rd123!'  // Hardcoded secret!
        AWS_ACCESS_KEY = 'AKIAIOSFODNN7EXAMPLE'
    }
    stages {
        stage('Deploy') {
            steps {
                sh '''
                    echo "Connecting with password: $DB_PASSWORD"
                    mysql -u admin -p$DB_PASSWORD -h prod-db.company.com
                '''
            }
        }
    }
}
```

Console logs show:
```
[Pipeline] echo
Connecting with password: prod_p@ssw0rd123!
```

The security team is demanding immediate remediation. You need to secure credentials across 200+ jobs.

## The Challenge

Implement a comprehensive credential management solution that secures all secrets, prevents exposure in logs, and integrates with enterprise secret management tools.


### Step 1: Use Jenkins Credentials Store Properly

```groovy
pipeline {
    agent any

    environment {
        // Bind credentials to environment variables - automatically masked
        DB_CREDS = credentials('production-db-credentials')
        // For username/password credentials, creates:
        // DB_CREDS_USR - username
        // DB_CREDS_PSW - password (masked in logs)
    }

    stages {
        stage('Deploy') {
            steps {
                // Password is automatically masked as ****
                sh '''
                    mysql -u $DB_CREDS_USR -p$DB_CREDS_PSW -h prod-db.company.com
                '''
            }
        }
    }
}
```

### Step 2: Use withCredentials for Scoped Access

```groovy
pipeline {
    agent any

    stages {
        stage('Deploy to AWS') {
            steps {
                // Credentials only available in this block
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws-credentials',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    ),
                    string(
                        credentialsId: 'deploy-token',
                        variable: 'DEPLOY_TOKEN'
                    ),
                    file(
                        credentialsId: 'kubeconfig-prod',
                        variable: 'KUBECONFIG'
                    )
                ]) {
                    sh '''
                        aws s3 cp dist/ s3://my-bucket/ --recursive
                        kubectl --kubeconfig=$KUBECONFIG apply -f k8s/
                    '''
                }
                // Credentials are no longer available here
            }
        }
    }
}
```

### Step 3: Integrate with HashiCorp Vault

```groovy
// Install: HashiCorp Vault Plugin
pipeline {
    agent any

    stages {
        stage('Deploy with Vault Secrets') {
            steps {
                withVault(
                    configuration: [
                        vaultUrl: 'https://vault.company.com',
                        vaultCredentialId: 'vault-approle'
                    ],
                    vaultSecrets: [
                        [
                            path: 'secret/data/production/database',
                            secretValues: [
                                [envVar: 'DB_HOST', vaultKey: 'host'],
                                [envVar: 'DB_USER', vaultKey: 'username'],
                                [envVar: 'DB_PASS', vaultKey: 'password']
                            ]
                        ],
                        [
                            path: 'secret/data/production/api-keys',
                            secretValues: [
                                [envVar: 'STRIPE_KEY', vaultKey: 'stripe_secret']
                            ]
                        ]
                    ]
                ) {
                    sh '''
                        # Secrets are available and automatically masked
                        ./deploy.sh --db-host=$DB_HOST --db-user=$DB_USER
                    '''
                }
            }
        }
    }
}
```

### Step 4: Implement Dynamic Secrets with Vault

```groovy
// vars/withDynamicDbCredentials.groovy
def call(Map config, Closure body) {
    def vaultPath = config.vaultPath ?: 'database/creds/app-role'
    def ttl = config.ttl ?: '1h'

    withVault(
        configuration: [
            vaultUrl: env.VAULT_ADDR,
            vaultCredentialId: 'vault-approle'
        ],
        vaultSecrets: [[
            path: vaultPath,
            secretValues: [
                [envVar: 'DB_USER', vaultKey: 'username'],
                [envVar: 'DB_PASS', vaultKey: 'password']
            ]
        ]]
    ) {
        echo "Using dynamic database credentials (TTL: ${ttl})"
        body()
    }
    // Credentials automatically expire after TTL
}

// Usage in pipeline
stage('Database Migration') {
    steps {
        script {
            withDynamicDbCredentials(ttl: '15m') {
                sh './run-migrations.sh'
            }
        }
    }
}
```

### Step 5: Prevent Secret Exposure in Logs

```groovy
pipeline {
    agent any

    options {
        // Mask any string matching these patterns
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Safe Logging') {
            steps {
                script {
                    // Wrap sensitive operations
                    wrap([$class: 'MaskPasswordsBuildWrapper']) {
                        withCredentials([string(credentialsId: 'api-key', variable: 'API_KEY')]) {
                            // Never echo secrets directly
                            sh '''
                                # Bad - might expose in error messages
                                # curl -H "Authorization: $API_KEY" https://api.example.com

                                # Good - use config file
                                echo "Authorization: $API_KEY" > .auth-header
                                curl -H @.auth-header https://api.example.com
                                rm -f .auth-header
                            '''
                        }
                    }
                }
            }
        }
    }
}
```

### Step 6: Secure Pipeline Pattern

```groovy
// vars/secureDeployment.groovy - Shared library function
def call(Map config) {
    def environment = config.environment
    def credentialPrefix = "deploy-${environment}"

    // Validate caller has permission
    def approvedTeams = ['platform', 'sre', 'devops']
    if (!approvedTeams.contains(config.team)) {
        error "Team ${config.team} is not authorized for deployments"
    }

    pipeline {
        agent { label 'secure-agents' }

        options {
            timeout(time: 30, unit: 'MINUTES')
        }

        stages {
            stage('Validate') {
                steps {
                    script {
                        // Verify credentials exist before proceeding
                        def requiredCreds = [
                            "${credentialPrefix}-kubeconfig",
                            "${credentialPrefix}-registry"
                        ]

                        requiredCreds.each { credId ->
                            try {
                                withCredentials([string(credentialsId: credId, variable: 'TEST')]) {
                                    echo "Credential ${credId} verified"
                                }
                            } catch (Exception e) {
                                error "Missing required credential: ${credId}"
                            }
                        }
                    }
                }
            }

            stage('Deploy') {
                steps {
                    withCredentials([
                        file(credentialsId: "${credentialPrefix}-kubeconfig", variable: 'KUBECONFIG'),
                        usernamePassword(
                            credentialsId: "${credentialPrefix}-registry",
                            usernameVariable: 'REGISTRY_USER',
                            passwordVariable: 'REGISTRY_PASS'
                        )
                    ]) {
                        sh '''
                            # Login to registry
                            echo $REGISTRY_PASS | docker login -u $REGISTRY_USER --password-stdin

                            # Deploy using kubeconfig
                            kubectl apply -f k8s/${ENVIRONMENT}/

                            # Cleanup
                            docker logout
                        '''
                    }
                }
            }
        }

        post {
            always {
                // Audit log
                script {
                    def auditEntry = [
                        timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ss'Z'"),
                        job: env.JOB_NAME,
                        build: env.BUILD_NUMBER,
                        user: currentBuild.getBuildCauses()[0]?.userId ?: 'system',
                        environment: environment,
                        result: currentBuild.result
                    ]
                    writeJSON file: 'audit.json', json: auditEntry
                    archiveArtifacts artifacts: 'audit.json'
                }
            }
        }
    }
}
```

### Step 7: Credential Rotation Script

```groovy
// Jenkinsfile for credential rotation
pipeline {
    agent any

    triggers {
        cron('0 0 1 * *')  // Monthly rotation
    }

    stages {
        stage('Rotate Credentials') {
            steps {
                script {
                    def credentialsToRotate = [
                        'production-db-password',
                        'staging-db-password',
                        'api-keys-production'
                    ]

                    credentialsToRotate.each { credId ->
                        echo "Rotating credential: ${credId}"

                        // Generate new secret
                        def newSecret = sh(
                            script: 'openssl rand -base64 32',
                            returnStdout: true
                        ).trim()

                        // Update in Vault
                        withVault(configuration: [vaultUrl: env.VAULT_ADDR, vaultCredentialId: 'vault-admin']) {
                            sh """
                                vault kv put secret/jenkins/${credId} value='${newSecret}'
                            """
                        }

                        echo "Rotated ${credId} successfully"
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#security-ops',
                message: "Credential rotation completed successfully"
            )
        }
    }
}
```


## Credential Security Checklist

| Risk | Mitigation | Implementation |
|------|-----------|----------------|
| Hardcoded secrets | Use credentials() binding | Replace all plaintext with credential IDs |
| Log exposure | MaskPasswordsBuildWrapper | Enable globally in Jenkins config |
| Broad access | Credential domains/folders | Scope credentials to specific folders |
| No rotation | Automated rotation jobs | Monthly rotation with Vault integration |
| No audit trail | Credential access logging | Enable audit plugin, send to SIEM |


---

### Quick Check

**What happens when you use the credentials() helper in Jenkins pipeline environment block?**

   A. The secret is stored in plaintext in the Jenkinsfile
   B. The secret is encrypted and stored in the build log
-> C. **The secret is bound to environment variables and automatically masked in logs**
   D. The secret is only available in the post section

<details>
<summary>See Answer</summary>

When using credentials() in the environment block, Jenkins binds the secret to environment variables and automatically masks the value in build logs. For username/password credentials, it creates three variables: CRED, CRED_USR, and CRED_PSW. The password variable is automatically masked - any occurrence in logs is replaced with ****. This is the recommended way to use credentials in pipelines.

</details>

---

### Question 5: Single Jenkins server is overloaded. Design a distributed build architecture with dynamic agents.

**Type:** Architecture | **Category:** Distributed Builds

## The Scenario

Your Jenkins master server is struggling:

```
Jenkins System Metrics:
- CPU: 95% (constant)
- Memory: 14GB / 16GB
- Build Queue: 47 builds waiting
- Average Wait Time: 25 minutes
- Executor Count: 8 (all busy)
```

Teams are complaining that builds take forever to start. The single server approach worked when you had 10 developers, but now you have 200. Management wants a solution that scales.

## The Challenge

Design a distributed Jenkins architecture with dynamic agents that can handle 500+ concurrent builds while keeping the master server stable and maintaining cost efficiency.


### Step 1: Architect the Distributed System

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Jenkins Distributed Architecture                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  ┌─────────────┐                                                      │
│  │   Jenkins   │  - No builds on master (executors = 0)              │
│  │   Master    │  - Manages jobs, credentials, plugins               │
│  │             │  - HA with standby or Kubernetes deployment         │
│  └──────┬──────┘                                                      │
│         │                                                             │
│         │ JNLP/SSH                                                    │
│         │                                                             │
│  ┌──────┴──────────────────────────────────────────────────────┐     │
│  │                    Agent Provisioners                        │     │
│  ├──────────────┬──────────────┬──────────────┬────────────────┤     │
│  │  Kubernetes  │   EC2/GCP    │   Docker     │  Static Agents │     │
│  │   Plugin     │   Plugin     │   Plugin     │  (Permanent)   │     │
│  └──────┬───────┴──────┬───────┴──────┬───────┴────────┬───────┘     │
│         │              │              │                │              │
│  ┌──────┴──────┐ ┌─────┴─────┐ ┌──────┴──────┐ ┌───────┴───────┐     │
│  │ K8s Pods    │ │ EC2       │ │ Docker      │ │ Bare Metal    │     │
│  │ (ephemeral) │ │ Instances │ │ Containers  │ │ Servers       │     │
│  └─────────────┘ └───────────┘ └─────────────┘ └───────────────┘     │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

### Step 2: Configure Master for Coordination Only

```groovy
// Configure via JCasC (jenkins.yaml)
jenkins:
  numExecutors: 0  # No builds on master
  mode: EXCLUSIVE

  nodes:
    - permanent:
        name: "static-agent-01"
        remoteFS: "/var/jenkins"
        launcher:
          ssh:
            host: "agent-01.company.com"
            credentialsId: "jenkins-ssh-key"
            sshHostKeyVerificationStrategy: "knownHostsFileKeyVerificationStrategy"
        nodeProperties:
          - toolLocation:
              locations:
                - key: "maven"
                  home: "/opt/maven"

  clouds:
    - kubernetes:
        name: "kubernetes"
        serverUrl: "https://kubernetes.default"
        jenkinsUrl: "http://jenkins.jenkins.svc.cluster.local:8080"
        jenkinsTunnel: "jenkins-agent.jenkins.svc.cluster.local:50000"
        containerCapStr: "100"
        maxRequestsPerHostStr: "32"
        templates:
          - name: "default"
            label: "kubernetes"
            nodeUsageMode: NORMAL
            containers:
              - name: "jnlp"
                image: "jenkins/inbound-agent:latest"
                resourceRequestCpu: "500m"
                resourceRequestMemory: "512Mi"
                resourceLimitCpu: "1"
                resourceLimitMemory: "1Gi"
```

### Step 3: Implement Kubernetes Dynamic Agents

```groovy
// Pipeline with Kubernetes pod template
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  serviceAccountName: jenkins-agent
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "200m"
      limits:
        memory: "512Mi"
        cpu: "500m"
  - name: maven
    image: maven:3.8-openjdk-17
    command: ['sleep', 'infinity']
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1"
    volumeMounts:
    - name: maven-cache
      mountPath: /root/.m2
  - name: docker
    image: docker:24-dind
    securityContext:
      privileged: true
    volumeMounts:
    - name: docker-graph
      mountPath: /var/lib/docker
  volumes:
  - name: maven-cache
    persistentVolumeClaim:
      claimName: maven-cache-pvc
  - name: docker-graph
    emptyDir: {}
'''
        }
    }

    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                container('docker') {
                    sh '''
                        docker build -t myapp:${BUILD_NUMBER} .
                        docker push registry.company.com/myapp:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }
}
```

### Step 4: Implement EC2 Dynamic Agents

```groovy
// JCasC configuration for EC2 plugin
jenkins:
  clouds:
    - amazonEC2:
        name: "aws-ec2"
        region: "us-east-1"
        credentialsId: "aws-credentials"
        sshKeysCredentialsId: "ec2-ssh-key"
        templates:
          - description: "Linux Build Agent"
            ami: "ami-0123456789abcdef0"
            type: M5Large
            remoteFS: "/var/jenkins"
            labels: "linux docker"
            mode: NORMAL
            numExecutors: 2
            securityGroups: "jenkins-agent-sg"
            subnetId: "subnet-abc123"
            associatePublicIp: false
            spotConfig:
              spotMaxBidPrice: "0.10"
              fallbackToOndemand: true
            idleTerminationMinutes: 15
            minimumNumberOfInstances: 0
            maximumNumberOfInstances: 50
            initScript: |
              #!/bin/bash
              yum update -y
              yum install -y docker java-17-amazon-corretto
              systemctl start docker
              usermod -aG docker ec2-user

          - description: "Windows Build Agent"
            ami: "ami-windows-server-2022"
            type: M5Large
            labels: "windows dotnet"
            mode: EXCLUSIVE
            numExecutors: 1
            minimumNumberOfInstances: 0
            maximumNumberOfInstances: 20
```

### Step 5: Implement Auto-Scaling Based on Queue

```groovy
// Shared library function for smart agent selection
// vars/smartAgent.groovy
def call(Map config = [:]) {
    def queueDepth = Jenkins.instance.queue.items.length
    def agentType = config.type ?: 'default'

    // Select agent based on queue depth and requirements
    if (agentType == 'heavy' || config.memory > '4Gi') {
        return kubernetes {
            yaml getLargeAgentPod(config)
        }
    } else if (queueDepth > 20) {
        // Use spot instances for cost savings during high load
        return label('spot-agents')
    } else {
        return kubernetes {
            yaml getDefaultAgentPod(config)
        }
    }
}

def getDefaultAgentPod(Map config) {
    return """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "${config.memory ?: '512Mi'}"
        cpu: "${config.cpu ?: '500m'}"
"""
}

def getLargeAgentPod(Map config) {
    return """
apiVersion: v1
kind: Pod
spec:
  nodeSelector:
    node-type: high-memory
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "${config.memory ?: '4Gi'}"
        cpu: "${config.cpu ?: '2'}"
      limits:
        memory: "${config.memoryLimit ?: '8Gi'}"
        cpu: "${config.cpuLimit ?: '4'}"
"""
}
```

### Step 6: Monitor and Auto-Scale

```groovy
// Monitoring pipeline for agent scaling
pipeline {
    agent { label 'monitoring' }

    triggers {
        cron('*/5 * * * *')  // Every 5 minutes
    }

    stages {
        stage('Check Queue Metrics') {
            steps {
                script {
                    def metrics = getJenkinsMetrics()

                    if (metrics.queueDepth > 30 && metrics.idleAgents < 5) {
                        echo "High queue depth (${metrics.queueDepth}), scaling up..."
                        scaleAgents(targetCount: metrics.queueDepth / 2)
                    }

                    if (metrics.queueDepth < 5 && metrics.idleAgents > 10) {
                        echo "Low queue depth, allowing scale down..."
                        // EC2 plugin handles termination based on idle time
                    }

                    // Alert if wait time is too high
                    if (metrics.avgWaitTime > 600) {  // 10 minutes
                        slackSend(
                            channel: '#jenkins-alerts',
                            message: "Build queue wait time is ${metrics.avgWaitTime}s"
                        )
                    }
                }
            }
        }
    }
}

def getJenkinsMetrics() {
    def jenkins = Jenkins.instance
    return [
        queueDepth: jenkins.queue.items.length,
        idleAgents: jenkins.nodes.findAll { it.computer.isIdle() }.size(),
        busyAgents: jenkins.nodes.findAll { !it.computer.isIdle() }.size(),
        avgWaitTime: jenkins.queue.items.collect {
            System.currentTimeMillis() - it.getInQueueSince()
        }.average() ?: 0
    ]
}
```


## Distributed Architecture Benefits

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Queue Wait Time | 25 min | 2 min | 92% reduction |
| Concurrent Builds | 8 | 100+ | 12x increase |
| Master CPU | 95% | 15% | Stable |
| Cost per Build | Fixed | Pay-per-use | 40% savings |
| Recovery Time | Hours | Minutes | HA enabled |

## Agent Selection Guide

| Workload | Agent Type | Why |
|----------|-----------|-----|
| Quick tests | Kubernetes pods | Fast startup, ephemeral |
| Docker builds | EC2 with Docker | DinD complexity in K8s |
| Windows builds | EC2 Windows AMI | Native environment |
| GPU workloads | EC2 GPU instances | Specialized hardware |
| Sensitive builds | Static agents | Network isolation |


---

### Quick Check

**Why should you set numExecutors to 0 on the Jenkins master in a distributed architecture?**

   A. To save disk space on the master
   B. Because the master doesn't support running builds
-> C. **To keep the master stable and responsive by dedicating it to coordination only**
   D. To reduce licensing costs

<details>
<summary>See Answer</summary>

Setting numExecutors to 0 on the master ensures it only handles coordination tasks: managing the queue, distributing builds to agents, storing configurations, and serving the UI. Running builds on the master consumes resources that could cause the master to become unresponsive, affecting all teams. A stable master is critical for a reliable CI/CD system, so all actual build work should run on agents.

</details>

---

### Question 6: Teams need automated CI for all branches with different deployment targets. Implement multi-branch pipelines.

**Type:** Practical | **Category:** Multi-branch Pipelines

## The Scenario

Your team has grown and adopted a branching strategy:
- `main` - Production deployments
- `develop` - Staging deployments
- `feature/*` - Dev environment + PR checks
- `hotfix/*` - Fast-track to production

Currently, you manually create a new job for each branch. With 50 active branches across 20 repositories, this is unsustainable. Teams want:
- Automatic pipeline creation for new branches
- Different behaviors per branch type
- PR status checks in GitHub
- Automatic cleanup when branches are deleted

## The Challenge

Implement a multi-branch pipeline architecture that automatically manages CI/CD for all branches with appropriate deployment targets and integrations.


### Step 1: Create Multi-branch Pipeline Job

```groovy
// Jenkinsfile in repository root
pipeline {
    agent any

    options {
        buildDiscarder(logRotator(
            numToKeepStr: env.BRANCH_NAME == 'main' ? '50' : '10'
        ))
        timestamps()
        timeout(time: 60, unit: 'MINUTES')
    }

    environment {
        APP_NAME = 'my-application'
        REGISTRY = 'registry.company.com'
        // Set based on branch
        DEPLOY_ENV = getBranchEnvironment()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = "${env.GIT_COMMIT_SHORT}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }

        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                }
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
            }
        }

        stage('Security Scan') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    changeRequest()  // PRs
                }
            }
            steps {
                sh 'npm audit --production'
                sh 'trivy fs --exit-code 1 --severity HIGH,CRITICAL .'
            }
        }

        stage('Build Docker Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    branch pattern: 'release/*', comparator: 'GLOB'
                }
            }
            steps {
                sh """
                    docker build -t ${REGISTRY}/${APP_NAME}:${IMAGE_TAG} .
                    docker push ${REGISTRY}/${APP_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to Dev') {
            when {
                branch pattern: 'feature/*', comparator: 'GLOB'
            }
            steps {
                deployTo(environment: 'dev', tag: env.IMAGE_TAG)
            }
        }

        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                deployTo(environment: 'staging', tag: env.IMAGE_TAG)
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // Require manual approval for production
                    timeout(time: 24, unit: 'HOURS') {
                        input message: 'Deploy to Production?',
                              ok: 'Deploy',
                              submitter: 'release-managers'
                    }
                }
                deployTo(environment: 'production', tag: env.IMAGE_TAG)
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/test-results/*.xml'
        }
        success {
            script {
                if (env.CHANGE_ID) {  // It's a PR
                    pullRequest.comment("Build passed! Image: `${REGISTRY}/${APP_NAME}:${IMAGE_TAG}`")
                }
            }
        }
        failure {
            script {
                if (env.BRANCH_NAME == 'main' || env.BRANCH_NAME == 'develop') {
                    slackSend(
                        channel: '#build-failures',
                        color: 'danger',
                        message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                    )
                }
            }
        }
    }
}

def getBranchEnvironment() {
    if (env.BRANCH_NAME == 'main') return 'production'
    if (env.BRANCH_NAME == 'develop') return 'staging'
    if (env.BRANCH_NAME?.startsWith('release/')) return 'staging'
    if (env.BRANCH_NAME?.startsWith('feature/')) return 'dev'
    if (env.BRANCH_NAME?.startsWith('hotfix/')) return 'production'
    return 'dev'
}

def deployTo(Map args) {
    def env = args.environment
    def tag = args.tag

    withCredentials([file(credentialsId: "kubeconfig-${env}", variable: 'KUBECONFIG')]) {
        sh """
            kubectl set image deployment/${APP_NAME} \
                app=${REGISTRY}/${APP_NAME}:${tag} \
                -n ${APP_NAME}-${env}

            kubectl rollout status deployment/${APP_NAME} \
                -n ${APP_NAME}-${env} \
                --timeout=300s
        """
    }
}
```

### Step 2: Configure Organization Folder

```groovy
// JCasC configuration for GitHub Organization scanning
jenkins:
  jobs:
    - script: >
        organizationFolder('my-org') {
          description('All repositories in my-org')
          displayName('My Organization')

          organizations {
            github {
              repoOwner('my-org')
              credentialsId('github-app')
              traits {
                gitHubBranchDiscovery {
                  strategyId(1)  // Exclude branches also filed as PRs
                }
                gitHubPullRequestDiscovery {
                  strategyId(1)  // Merge PR with target branch
                }
                gitHubForkDiscovery {
                  strategyId(1)
                  trust {
                    gitHubTrustPermissions()
                  }
                }
              }
            }
          }

          projectFactories {
            workflowMultiBranchProjectFactory {
              scriptPath('Jenkinsfile')
            }
          }

          buildStrategies {
            buildRegularBranches()
            buildChangeRequests {
              ignoreTargetOnlyChanges(false)
            }
            buildTags {
              atLeastDays('-1')
              atMostDays('7')
            }
          }

          orphanedItemStrategy {
            discardOldItems {
              daysToKeep(7)
              numToKeep(20)
            }
          }

          triggers {
            periodicFolderTrigger {
              interval('1h')
            }
          }
        }
```

### Step 3: GitHub Integration for PR Status

```groovy
// Enhanced Jenkinsfile with GitHub status updates
pipeline {
    agent any

    options {
        // Report build status to GitHub
        githubProjectProperty(projectUrlStr: 'https://github.com/my-org/my-app')
    }

    stages {
        stage('PR Validation') {
            when {
                changeRequest()
            }
            steps {
                script {
                    // Validate PR requirements
                    def prInfo = pullRequest

                    // Check PR size
                    def changedFiles = prInfo.changedFiles
                    if (changedFiles > 50) {
                        echo "WARNING: Large PR with ${changedFiles} files"
                    }

                    // Check for required labels
                    def labels = prInfo.labels.collect { it.toLowerCase() }
                    if (!labels.any { it in ['bug', 'feature', 'enhancement', 'docs'] }) {
                        error 'PR must have a type label (bug, feature, enhancement, or docs)'
                    }

                    // Add reviewer suggestions based on files changed
                    if (prInfo.files.any { it.path.startsWith('security/') }) {
                        prInfo.addLabel('needs-security-review')
                        prInfo.comment('@security-team - Please review security changes')
                    }
                }
            }
        }

        stage('Build & Test') {
            steps {
                // GitHub sees each stage as a separate check
                script {
                    setBuildStatus('pending', 'Build started')
                }
                sh 'npm ci && npm run build && npm test'
                script {
                    setBuildStatus('success', 'Build passed')
                }
            }
        }

        stage('Integration Tests') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                    changeRequest target: 'main'
                    changeRequest target: 'develop'
                }
            }
            steps {
                sh 'npm run test:integration'
            }
        }
    }

    post {
        success {
            script {
                if (env.CHANGE_ID) {
                    // Update PR with success status
                    setBuildStatus('success', 'All checks passed')

                    // Add deployment preview URL for feature branches
                    def previewUrl = "https://${env.CHANGE_ID}.preview.company.com"
                    pullRequest.comment("""
                        Build successful!

                        **Preview URL:** ${previewUrl}
                        **Image Tag:** `${env.IMAGE_TAG}`
                    """)
                }
            }
        }
        failure {
            script {
                if (env.CHANGE_ID) {
                    setBuildStatus('failure', 'Build failed')
                }
            }
        }
    }
}

def setBuildStatus(String state, String description) {
    step([
        $class: 'GitHubCommitStatusSetter',
        reposSource: [$class: 'ManuallyEnteredRepositorySource', url: env.GIT_URL],
        commitShaSource: [$class: 'ManuallyEnteredShaSource', sha: env.GIT_COMMIT],
        contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'ci/jenkins'],
        statusResultSource: [$class: 'ConditionalStatusResultSource', results: [
            [$class: 'AnyBuildResult', state: state, message: description]
        ]]
    ])
}
```

### Step 4: Branch-Specific Behaviors

```groovy
// More complex branch logic
pipeline {
    agent any

    stages {
        stage('Initialize') {
            steps {
                script {
                    // Determine build configuration based on branch
                    env.RUN_E2E = shouldRunE2E()
                    env.DEPLOY_ENABLED = shouldDeploy()
                    env.SONAR_ENABLED = shouldRunSonar()
                }
            }
        }

        stage('E2E Tests') {
            when {
                expression { env.RUN_E2E == 'true' }
            }
            steps {
                sh 'npm run test:e2e'
            }
        }

        stage('SonarQube Analysis') {
            when {
                expression { env.SONAR_ENABLED == 'true' }
            }
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'npm run sonar'
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}

def shouldRunE2E() {
    // Run E2E for main branches and PRs targeting them
    if (env.BRANCH_NAME in ['main', 'develop']) return true
    if (env.CHANGE_TARGET in ['main', 'develop']) return true
    return false
}

def shouldDeploy() {
    // Only deploy from specific branches
    def deployBranches = ['main', 'develop', ~/release\/.*/]
    return deployBranches.any { pattern ->
        if (pattern instanceof java.util.regex.Pattern) {
            return env.BRANCH_NAME ==~ pattern
        }
        return env.BRANCH_NAME == pattern
    }
}

def shouldRunSonar() {
    // Run SonarQube on main branches and weekly on features
    if (env.BRANCH_NAME in ['main', 'develop']) return true
    if (env.BRANCH_NAME?.startsWith('feature/')) {
        // Only on scheduled builds, not every push
        return currentBuild.getBuildCauses('hudson.triggers.TimerTrigger$TimerTriggerCause')
    }
    return false
}
```


## Multi-branch Pipeline Matrix

| Branch Pattern | Build | Test | Deploy | Approval | Cleanup |
|---------------|-------|------|--------|----------|---------|
| main | Full | All | Production | Required | Never |
| develop | Full | All | Staging | Auto | Never |
| release/* | Full | All | Staging | Required | After merge |
| feature/* | Quick | Unit | Dev | Auto | 7 days |
| hotfix/* | Full | All | Production | Required | After merge |
| PR to main | Full | All | Preview | N/A | After close |

---

### Question 7: Jenkins agents keep disconnecting and builds are stuck in queue. Diagnose and fix the issue.

**Type:** Debugging | **Category:** Agent Management

## The Scenario

Your Jenkins build queue is growing rapidly:

```
Build Queue (47 items):
  - deploy-production #234 (waiting for agent 'linux-docker')
  - unit-tests #1892 (waiting for 25 minutes)
  - integration-tests #445 (pending - No matching agents)
  ...

Agent Status:
  - linux-docker-01: Offline (connection reset)
  - linux-docker-02: Offline (ping timeout)
  - linux-docker-03: Connected (launching...)
  - kubernetes-pod-xyz: Offline (terminated)
```

Developers are escalating. The agents were working fine yesterday. You need to diagnose and fix this quickly.

## The Challenge

Systematically debug the agent connectivity issues, identify root causes, and implement fixes along with monitoring to prevent recurrence.


### Step 1: Quick Diagnostic Commands

```bash
# Check Jenkins master logs for agent errors
tail -f /var/log/jenkins/jenkins.log | grep -i "agent\|slave\|connect"

# Check agent status via API
curl -s http://jenkins:8080/computer/api/json?pretty=true \
    -u admin:$TOKEN | jq '.computer[] | {name: .displayName, offline: .offline, offlineCauseReason: .offlineCauseReason}'

# Check network connectivity to agents
for agent in agent-01 agent-02 agent-03; do
    echo "=== $agent ==="
    ping -c 2 $agent
    nc -zv $agent 22    # SSH
    nc -zv $agent 50000 # JNLP
done

# Check if agents can reach master
ssh agent-01 "curl -s http://jenkins:8080/login | head -5"
ssh agent-01 "nc -zv jenkins 8080 && nc -zv jenkins 50000"
```

### Step 2: Check Agent Logs

```bash
# On SSH-connected agents
ssh agent-01 "tail -100 /var/log/jenkins/agent.log"

# Common errors to look for:
# - "java.net.SocketTimeoutException" - Network issue
# - "java.lang.OutOfMemoryError" - Agent JVM memory
# - "hudson.remoting.ChannelClosedException" - Connection dropped
# - "java.security.cert.CertificateException" - SSL issues
```

```groovy
// Check agent status programmatically in Jenkins Script Console


Jenkins.instance.computers.each { computer ->
    if (computer.name != "") {
        println "=== ${computer.name} ==="
        println "  Online: ${computer.isOnline()}"
        println "  Connecting: ${computer.isConnecting()}"
        if (computer.isOffline()) {
            println "  Offline Cause: ${computer.getOfflineCauseReason()}"
            println "  Offline Since: ${computer.getOfflineCause()?.getTimestamp()}"
        }
        println "  Executors: ${computer.numExecutors}"
        println "  Busy Executors: ${computer.countBusy()}"
        println ""
    }
}
```

### Step 3: Common Issues and Fixes

```groovy
// Fix 1: Agent JVM memory issues
// In agent launch command, increase heap size
// Before: java -jar agent.jar
// After:
java -Xmx2g -Xms512m \
     -XX:+UseG1GC \
     -XX:MaxGCPauseMillis=200 \
     -jar agent.jar \
     -jnlpUrl http://jenkins:8080/computer/agent-01/slave-agent.jnlp \
     -secret @secret-file

// Fix 2: Network timeout issues - increase timeouts
// In Jenkins > Configure Global Security
// Agent -> Controller Security:
//   - TCP port for inbound agents: Fixed (50000)
//   - Agent protocols: Enable "Inbound TCP Agent Protocol/4 (TLS encryption)"
```

```groovy
// Fix 3: Reconnection settings for SSH agents
// Configure via JCasC
jenkins:
  nodes:
    - permanent:
        name: "linux-agent-01"
        remoteFS: "/var/jenkins"
        launcher:
          ssh:
            host: "agent-01.company.com"
            credentialsId: "jenkins-ssh"
            maxNumRetries: 5
            retryWaitTime: 30
            sshHostKeyVerificationStrategy:
              manuallyTrustedKeyVerificationStrategy:
                requireInitialManualTrust: false
            javaPath: "/usr/bin/java"
            jvmOptions: "-Xmx1g"
        nodeProperties:
          - diskSpaceMonitor:
              freeDiskSpaceThreshold: "5GB"
              freeDiskSpaceWarningThreshold: "10GB"
        retentionStrategy:
          demand:
            idleDelay: 10
            inDemandDelay: 0
```

### Step 4: Kubernetes Agent Issues

```groovy
// Debug Kubernetes agent issues
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "512Mi"
        cpu: "500m"
      limits:
        memory: "1Gi"
        cpu: "1"
    # Add liveness probe to detect hung agents
    livenessProbe:
      httpGet:
        path: /tcpSlaveAgentListener/
        port: 50000
      initialDelaySeconds: 60
      periodSeconds: 30
'''
        }
    }

    stages {
        stage('Debug') {
            steps {
                sh '''
                    echo "=== Pod Info ==="
                    hostname
                    cat /etc/resolv.conf

                    echo "=== Jenkins Master Connectivity ==="
                    nc -zv $JENKINS_URL 8080 || true
                    nc -zv $JENKINS_TUNNEL_HOST 50000 || true

                    echo "=== DNS Resolution ==="
                    nslookup jenkins.jenkins.svc.cluster.local || true
                '''
            }
        }
    }
}
```

```bash
# Kubernetes debugging commands
# Check pod status
kubectl get pods -n jenkins -l jenkins=agent

# Check pod events
kubectl describe pod <pod-name> -n jenkins | grep -A 20 "Events:"

# Check if service is accessible
kubectl run debug --rm -it --image=busybox -n jenkins -- sh
  nslookup jenkins.jenkins.svc.cluster.local
  nc -zv jenkins 8080
  nc -zv jenkins-agent 50000
```

### Step 5: Implement Health Monitoring

```groovy
// Jenkinsfile for agent health monitoring
pipeline {
    agent { label 'monitoring' }

    triggers {
        cron('*/5 * * * *')  // Every 5 minutes
    }

    stages {
        stage('Check Agent Health') {
            steps {
                script {
                    def unhealthyAgents = []

                    Jenkins.instance.computers.each { computer ->
                        if (computer.name != "" && computer.name != "master") {
                            def health = checkAgentHealth(computer)
                            if (!health.healthy) {
                                unhealthyAgents.add([
                                    name: computer.name,
                                    reason: health.reason
                                ])

                                // Attempt auto-recovery
                                if (health.recoverable) {
                                    echo "Attempting to reconnect ${computer.name}..."
                                    computer.connect(true)
                                }
                            }
                        }
                    }

                    if (unhealthyAgents) {
                        def message = "Unhealthy Jenkins Agents:\n"
                        unhealthyAgents.each { agent ->
                            message += "- ${agent.name}: ${agent.reason}\n"
                        }

                        slackSend(
                            channel: '#jenkins-alerts',
                            color: 'warning',
                            message: message
                        )
                    }
                }
            }
        }
    }
}

def checkAgentHealth(computer) {
    def result = [healthy: true, reason: '', recoverable: false]

    // Check if offline
    if (computer.isOffline()) {
        result.healthy = false
        result.reason = computer.getOfflineCauseReason() ?: 'Unknown'
        result.recoverable = !computer.isTemporarilyOffline()
        return result
    }

    // Check disk space
    def diskSpace = computer.getMonitorData()?.get('hudson.node_monitors.DiskSpaceMonitor')
    if (diskSpace && diskSpace.size < 5 * 1024 * 1024 * 1024) {  // Less than 5GB
        result.healthy = false
        result.reason = "Low disk space: ${diskSpace.size / 1024 / 1024 / 1024}GB"
        return result
    }

    // Check response time
    def responseTime = computer.getMonitorData()?.get('hudson.node_monitors.ResponseTimeMonitor')
    if (responseTime && responseTime.average > 5000) {  // More than 5 seconds
        result.healthy = false
        result.reason = "High response time: ${responseTime.average}ms"
        return result
    }

    return result
}
```

### Step 6: Preventive Configuration

```groovy
// JCasC configuration with resilient agent setup
jenkins:
  nodes:
    - permanent:
        name: "resilient-agent"
        remoteFS: "/var/jenkins"
        numExecutors: 4
        launcher:
          ssh:
            host: "agent.company.com"
            credentialsId: "jenkins-ssh"
            maxNumRetries: 10
            retryWaitTime: 60
            launchTimeoutSeconds: 300
        nodeProperties:
          - diskSpaceMonitor:
              freeDiskSpaceThreshold: "5GiB"
          - responseTimeMonitor:
              averageResponseTimeThreshold: 5000
          - clockMonitor:
              clockTimeDiffThreshold: 60000

  # Global agent settings
  remotingSecurity:
    enabled: true

unclassified:
  location:
    url: "https://jenkins.company.com/"

  # Agent controller connectivity
  slaveAgentPort: 50000
  agentProtocols:
    - "JNLP4-connect"
    - "Ping"
```


## Agent Troubleshooting Reference

| Symptom | Likely Cause | Diagnostic Command | Fix |
|---------|--------------|-------------------|-----|
| "Connection reset" | Network firewall | `nc -zv agent 50000` | Open port 50000 |
| "Ping timeout" | Agent process crashed | `ssh agent "ps aux \| grep java"` | Restart agent |
| "Channel closed" | JVM OOM | Check `/var/log/jenkins/agent.log` | Increase heap |
| "Certificate error" | SSL mismatch | `openssl s_client -connect jenkins:443` | Update certs |
| "Permission denied" | SSH key issues | `ssh -vvv agent` | Regenerate keys |

---

### Question 8: Migrate from static VMs to Kubernetes-based dynamic agents for cloud-native CI/CD.

**Type:** Practical | **Category:** Kubernetes Integration

## The Scenario

Your organization runs Jenkins on 20 static EC2 instances costing $15,000/month. You observe:

```
Current Infrastructure:
- 20 x m5.xlarge instances ($0.192/hr each)
- Average utilization: 35%
- Peak utilization: 100% (queue builds up)
- Off-hours utilization: 5%
- Monthly cost: ~$15,000
```

Leadership wants to reduce costs while improving scalability. The platform team has a production Kubernetes cluster available.

## The Challenge

Migrate Jenkins to Kubernetes with dynamic pod-based agents that scale to zero during off-hours and handle peak loads without queuing.


### Step 1: Deploy Jenkins Master on Kubernetes

```yaml
# jenkins-master-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      serviceAccountName: jenkins
      securityContext:
        fsGroup: 1000
        runAsUser: 1000
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts-jdk17
          ports:
            - containerPort: 8080
              name: http
            - containerPort: 50000
              name: jnlp
          resources:
            requests:
              memory: "2Gi"
              cpu: "1"
            limits:
              memory: "4Gi"
              cpu: "2"
          env:
            - name: JAVA_OPTS
              value: >-
                -Xmx2g
                -XX:+UseG1GC
                -Djenkins.install.runSetupWizard=false
            - name: CASC_JENKINS_CONFIG
              value: /var/jenkins_home/casc_configs
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
            - name: casc-config
              mountPath: /var/jenkins_home/casc_configs
          livenessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 120
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            periodSeconds: 10
      volumes:
        - name: jenkins-home
          persistentVolumeClaim:
            claimName: jenkins-home-pvc
        - name: casc-config
          configMap:
            name: jenkins-casc-config
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: jenkins
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
      name: http
    - port: 50000
      targetPort: 50000
      name: jnlp
  selector:
    app: jenkins
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-agent
  namespace: jenkins
spec:
  type: ClusterIP
  ports:
    - port: 50000
      targetPort: 50000
  selector:
    app: jenkins
```

### Step 2: Configure RBAC for Dynamic Agents

```yaml
# jenkins-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: jenkins
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: jenkins-agent-role
  namespace: jenkins
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["create", "delete", "get", "list", "watch", "patch"]
  - apiGroups: [""]
    resources: ["pods/exec"]
    verbs: ["create", "get"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get", "list"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["create", "delete", "get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: jenkins-agent-binding
  namespace: jenkins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: jenkins-agent-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: jenkins
```

### Step 3: Configure Kubernetes Cloud in JCasC

```yaml
# jenkins-casc-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: jenkins-casc-config
  namespace: jenkins
data:
  jenkins.yaml: |
    jenkins:
      numExecutors: 0
      mode: EXCLUSIVE
      clouds:
        - kubernetes:
            name: "kubernetes"
            serverUrl: "https://kubernetes.default"
            namespace: "jenkins"
            jenkinsUrl: "http://jenkins.jenkins.svc.cluster.local:8080"
            jenkinsTunnel: "jenkins-agent.jenkins.svc.cluster.local:50000"
            containerCapStr: "50"
            maxRequestsPerHostStr: "32"
            retentionTimeout: 5
            waitForPodSec: 600

            templates:
              # Default lightweight agent
              - name: "default"
                label: "kubernetes default"
                nodeUsageMode: "NORMAL"
                idleMinutes: 10
                containers:
                  - name: "jnlp"
                    image: "jenkins/inbound-agent:latest"
                    workingDir: "/home/jenkins/agent"
                    resourceRequestCpu: "200m"
                    resourceRequestMemory: "256Mi"
                    resourceLimitCpu: "500m"
                    resourceLimitMemory: "512Mi"
                yamlMergeStrategy: "override"

              # Node.js build agent
              - name: "nodejs"
                label: "nodejs npm"
                containers:
                  - name: "jnlp"
                    image: "jenkins/inbound-agent:latest"
                    resourceRequestMemory: "256Mi"
                    resourceLimitMemory: "512Mi"
                  - name: "node"
                    image: "node:18-alpine"
                    command: "sleep"
                    args: "infinity"
                    resourceRequestMemory: "1Gi"
                    resourceLimitMemory: "2Gi"
                volumes:
                  - hostPathVolume:
                      hostPath: "/var/run/docker.sock"
                      mountPath: "/var/run/docker.sock"

              # Docker build agent
              - name: "docker"
                label: "docker"
                containers:
                  - name: "jnlp"
                    image: "jenkins/inbound-agent:latest"
                  - name: "docker"
                    image: "docker:24-dind"
                    privileged: true
                    resourceRequestMemory: "1Gi"
                    resourceLimitMemory: "4Gi"
                volumes:
                  - emptyDirVolume:
                      mountPath: "/var/lib/docker"
                      memory: false

              # Heavy workload agent
              - name: "heavy"
                label: "heavy build"
                nodeSelector: "node-type=compute-optimized"
                containers:
                  - name: "jnlp"
                    image: "jenkins/inbound-agent:latest"
                    resourceRequestCpu: "2"
                    resourceRequestMemory: "4Gi"
                    resourceLimitCpu: "4"
                    resourceLimitMemory: "8Gi"
```

### Step 4: Create Optimized Pipeline with Kubernetes Agents

```groovy
// Jenkinsfile using Kubernetes agents
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-agent
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
  - name: node
    image: node:18-alpine
    command: ["sleep", "infinity"]
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1"
    volumeMounts:
    - name: npm-cache
      mountPath: /root/.npm
  - name: docker
    image: docker:24-cli
    command: ["sleep", "infinity"]
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
  - name: dind
    image: docker:24-dind
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
  volumes:
  - name: npm-cache
    persistentVolumeClaim:
      claimName: npm-cache-pvc
'''
        }
    }

    stages {
        stage('Build') {
            steps {
                container('node') {
                    sh '''
                        npm ci --cache /root/.npm
                        npm run build
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                container('node') {
                    sh 'npm test'
                }
            }
        }

        stage('Docker Build') {
            steps {
                container('docker') {
                    sh '''
                        docker build -t myapp:${BUILD_NUMBER} .
                        docker push registry.company.com/myapp:${BUILD_NUMBER}
                    '''
                }
            }
        }
    }
}
```

### Step 5: Implement Cache for Faster Builds

```yaml
# npm-cache-pvc.yaml - Shared cache for faster builds
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: npm-cache-pvc
  namespace: jenkins
spec:
  accessModes:
    - ReadWriteMany  # Multiple pods can share
  storageClassName: efs-sc  # Use EFS for shared storage
  resources:
    requests:
      storage: 50Gi
---
# maven-cache-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: maven-cache-pvc
  namespace: jenkins
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 100Gi
```

### Step 6: Implement Pod Template Selection Logic

```groovy
// vars/selectAgent.groovy - Shared library for smart agent selection
def call(Map config = [:]) {
    def workloadType = config.type ?: 'default'
    def resources = config.resources ?: [:]

    def podTemplates = [
        'default': defaultPod(),
        'nodejs': nodejsPod(resources),
        'java': javaPod(resources),
        'docker': dockerPod(resources),
        'heavy': heavyPod(resources)
    ]

    return kubernetes {
        yaml podTemplates[workloadType]
    }
}

def defaultPod() {
    return '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    resources:
      requests:
        memory: "256Mi"
        cpu: "100m"
'''
}

def nodejsPod(Map resources) {
    def memory = resources.memory ?: '1Gi'
    def cpu = resources.cpu ?: '500m'

    return """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
  - name: node
    image: node:18
    command: ["sleep", "infinity"]
    resources:
      requests:
        memory: "${memory}"
        cpu: "${cpu}"
      limits:
        memory: "${memory}"
        cpu: "${cpu}"
    volumeMounts:
    - name: npm-cache
      mountPath: /root/.npm
  volumes:
  - name: npm-cache
    persistentVolumeClaim:
      claimName: npm-cache-pvc
"""
}

// Usage in Jenkinsfile
pipeline {
    agent {
        script {
            selectAgent(
                type: 'nodejs',
                resources: [memory: '2Gi', cpu: '1']
            )
        }
    }
    stages {
        // ...
    }
}
```


## Migration Cost Comparison

| Metric | Before (VMs) | After (K8s) | Savings |
|--------|--------------|-------------|---------|
| Monthly Cost | $15,000 | $3,500 | 77% |
| Peak Capacity | 20 agents | 50+ pods | 150%+ |
| Off-hours Cost | $5,000 | $500 | 90% |
| Queue Wait Time | 15 min | 2 min | 87% |
| Setup Time | Hours | Seconds | 99% |


---

### Quick Check

**Why should you use a separate container for the build tools (like node, maven) instead of installing them in the jnlp container?**

   A. The jnlp container doesn't support additional tools
-> B. **It allows using different tool versions without rebuilding the agent image**
   C. Kubernetes doesn't allow single-container pods
   D. It reduces the pod startup time

<details>
<summary>See Answer</summary>

Using separate containers (sidecar pattern) for build tools allows you to use different versions of tools (Node 16, 18, 20) without maintaining multiple agent images. Each pipeline can specify the exact version it needs. The jnlp container handles Jenkins communication while tool containers handle the actual work. This separation of concerns makes pipelines more maintainable and flexible.

</details>

---

### Question 9: Everyone has admin access to Jenkins. Implement proper RBAC with team-based permissions.

**Type:** Practical | **Category:** Security & RBAC

## The Scenario

During a security audit, you discover that all 200 developers have admin access to Jenkins:

```
Current State:
- 200 users with admin access
- Any developer can:
  - View/modify all jobs
  - Access all credentials
  - Install/remove plugins
  - Modify global configuration
  - Delete production jobs

Recent Incidents:
- Developer accidentally deleted production deployment job
- Credentials exposed when someone added debug output
- Unauthorized plugin installation caused security vulnerability
```

Compliance requires least-privilege access, audit logging, and separation of duties.

## The Challenge

Implement a role-based access control system that gives teams access only to their own resources while maintaining operational efficiency.


### Step 1: Install Required Plugins

```groovy
// Required plugins for RBAC
// Install via Plugin Manager or JCasC
plugins:
  - role-strategy          # Role-based authorization
  - folder-auth            # Folder-level permissions
  - audit-trail            # Audit logging
  - oic-auth               # OpenID Connect (optional)
  - saml                   # SAML SSO (optional)
  - ldap                   # LDAP integration (optional)
```

### Step 2: Configure Folder-Based Authorization

```yaml
# JCasC configuration for Role Strategy Plugin
jenkins:
  authorizationStrategy:
    roleBased:
      roles:
        global:
          # Platform/Admin team - full access
          - name: "admin"
            permissions:
              - "Overall/Administer"
            entries:
              - group: "platform-admins"

          # Read-only for all authenticated users
          - name: "authenticated"
            permissions:
              - "Overall/Read"
              - "Job/Read"
              - "View/Read"
            entries:
              - group: "authenticated"

          # Ability to see agents (for debugging)
          - name: "agent-viewer"
            permissions:
              - "Agent/Connect"
              - "Agent/Build"
            entries:
              - group: "developers"

        items:
          # Development team permissions
          - name: "developer"
            permissions:
              - "Job/Build"
              - "Job/Cancel"
              - "Job/Read"
              - "Job/Workspace"
              - "Run/Replay"
              - "Run/Update"
            entries:
              - group: "developers"
            pattern: ".*"

          # Release manager permissions
          - name: "release-manager"
            permissions:
              - "Job/Build"
              - "Job/Cancel"
              - "Job/Configure"
              - "Job/Create"
              - "Job/Delete"
              - "Job/Read"
              - "Job/Workspace"
              - "Run/Delete"
              - "Run/Replay"
              - "Run/Update"
              - "Credentials/View"
            entries:
              - group: "release-managers"
            pattern: ".*"

        agents:
          - name: "agent-admin"
            permissions:
              - "Agent/Build"
              - "Agent/Configure"
              - "Agent/Connect"
              - "Agent/Create"
              - "Agent/Delete"
              - "Agent/Disconnect"
            entries:
              - group: "platform-admins"
```

### Step 3: Implement Folder-Based Team Isolation

```groovy
// Create folder structure via Job DSL
folder('teams') {
    displayName('Teams')
    description('Team-specific jobs')
}

['payments', 'platform', 'mobile', 'data-engineering'].each { team ->
    folder("teams/${team}") {
        displayName(team.capitalize())
        description("Jobs for ${team} team")

        // Team-specific authorization
        properties {
            authorizationMatrix {
                // Team members can build and view
                permissions([
                    "Job/Build:${team}-developers",
                    "Job/Cancel:${team}-developers",
                    "Job/Read:${team}-developers",
                    "Job/Workspace:${team}-developers",
                    "Run/Replay:${team}-developers",
                ])

                // Team leads can configure
                permissions([
                    "Job/Configure:${team}-leads",
                    "Job/Create:${team}-leads",
                    "Job/Delete:${team}-leads",
                    "Credentials/View:${team}-leads",
                ])

                // Platform team has access everywhere
                permissions([
                    "Job/Build:platform-admins",
                    "Job/Configure:platform-admins",
                    "Job/Create:platform-admins",
                    "Job/Delete:platform-admins",
                ])
            }
        }
    }
}
```

### Step 4: Configure LDAP/SSO Integration

```yaml
# JCasC LDAP configuration
jenkins:
  securityRealm:
    ldap:
      configurations:
        - server: "ldaps://ldap.company.com"
          rootDN: "dc=company,dc=com"
          userSearchBase: "ou=users"
          userSearchFilter: "uid={0}"
          groupSearchBase: "ou=groups"
          groupSearchFilter: "(member={0})"
          groupMembershipStrategy:
            fromGroupSearch:
              filter: "(&(cn={0})(objectclass=groupOfNames))"
          managerDN: "cn=jenkins,ou=service-accounts,dc=company,dc=com"
          managerPasswordSecret: "${LDAP_MANAGER_PASSWORD}"
      cache:
        size: 100
        ttl: 300
```

```yaml
# Alternative: SAML SSO configuration
jenkins:
  securityRealm:
    saml:
      displayNameAttribute: "displayName"
      emailAttribute: "email"
      groupsAttribute: "groups"
      idpMetadataConfiguration:
        url: "https://idp.company.com/metadata"
      maximumAuthenticationLifetime: 86400
      usernameCaseConversion: "lowercase"
```

### Step 5: Implement Credential Access Control

```yaml
# JCasC credential domains and folder-scoped credentials
credentials:
  system:
    domainCredentials:
      - domain:
          name: "production"
          description: "Production environment credentials"
          specifications:
            - hostnameSpecification:
                includes: "*.prod.company.com"
        credentials:
          - usernamePassword:
              id: "prod-deploy-creds"
              username: "deployer"
              password: "${PROD_DEPLOY_PASSWORD}"
              scope: GLOBAL
              description: "Production deployment credentials"

      - domain:
          name: "staging"
          description: "Staging environment credentials"
          specifications:
            - hostnameSpecification:
                includes: "*.staging.company.com"
        credentials:
          - usernamePassword:
              id: "staging-deploy-creds"
              username: "deployer"
              password: "${STAGING_DEPLOY_PASSWORD}"

# Folder-scoped credentials in job DSL
folder('teams/payments') {
    properties {
        folderCredentialsProvider {
            domainCredentials {
                domainCredentials {
                    domain {
                        name: "payments-secrets"
                    }
                    credentials {
                        usernamePassword {
                            id: "payments-db-creds"
                            username: "payments_app"
                            password: "${PAYMENTS_DB_PASSWORD}"
                            scope: "GLOBAL"
                        }
                    }
                }
            }
        }
    }
}
```

### Step 6: Implement Audit Logging

```yaml
# JCasC audit trail configuration
unclassified:
  auditTrail:
    logBuildCause: true
    pattern: ".*/(?:configSubmit|doDelete|postBuildResult|enable|disable|cancelQueue|stop|toggleLogKeep|doWipeOutWorkspace|createItem|createView|toggleOffline|cancelQuietDown|quietDown|restart|safeRestart|safeExit|exit|doDisconnect|launchSlaveAgent|doCreateItem|doCreateView)/?.*"
    loggers:
      - syslog:
          syslogHost: "syslog.company.com"
          facility: "USER"
          messageHostname: "jenkins-master"
      - file:
          log: "/var/log/jenkins/audit.log"
          limit: 100
          count: 10

  # Security configuration
  globalConfigFiles:
    configs:
      - json:
          id: "security-config"
          name: "Security Configuration"
          content: |
            {
              "sessionTimeout": 480,
              "disableRememberMe": true,
              "preventCSRF": true
            }
```

### Step 7: Create Permission Validation Pipeline

```groovy
// Pipeline to validate and report on permissions
pipeline {
    agent any

    triggers {
        cron('0 8 * * 1')  // Weekly Monday 8 AM
    }

    stages {
        stage('Audit Permissions') {
            steps {
                script {
                    def report = generatePermissionReport()
                    writeFile file: 'permission-report.html', text: report

                    // Check for violations
                    def violations = checkForViolations()
                    if (violations) {
                        emailext(
                            to: 'security-team@company.com',
                            subject: 'Jenkins Permission Violations Detected',
                            body: violations,
                            attachmentsPattern: 'permission-report.html'
                        )
                    }
                }
            }
        }
    }
}

def generatePermissionReport() {
    def report = new StringBuilder()
    report.append("<html><body><h1>Jenkins Permission Report</h1>")

    // List all users with admin access
    def authStrategy = Jenkins.instance.authorizationStrategy
    if (authStrategy instanceof RoleBasedAuthorizationStrategy) {
        def roleMap = authStrategy.getRoleMap(RoleBasedAuthorizationStrategy.GLOBAL)
        roleMap.each { role, sids ->
            report.append("<h2>Role: ${role.name}</h2>")
            report.append("<ul>")
            sids.each { sid ->
                report.append("<li>${sid}</li>")
            }
            report.append("</ul>")
        }
    }

    report.append("</body></html>")
    return report.toString()
}

def checkForViolations() {
    def violations = []

    // Check for users with excessive permissions
    Jenkins.instance.allItems(Folder.class).each { folder ->
        def props = folder.properties.get(AuthorizationMatrixProperty.class)
        if (props) {
            props.getGroups().each { sid ->
                def perms = props.getGrantedPermissions()
                if (perms.find { it.key.name.contains("Administer") }) {
                    violations.add("${sid} has admin access to folder: ${folder.fullName}")
                }
            }
        }
    }

    return violations ? violations.join("\n") : null
}
```


## RBAC Best Practices

| Principle | Implementation | Why |
|-----------|----------------|-----|
| Least Privilege | Start with no access, add as needed | Reduces blast radius of compromises |
| Team Isolation | Folder-based permissions | Teams can't affect each other |
| Credential Scoping | Folder credentials | Teams only see their secrets |
| Audit Trail | Log all permission changes | Compliance and forensics |
| Regular Review | Weekly permission audits | Catch permission creep |

---

### Question 10: Jenkins configuration is manual and inconsistent. Implement Configuration as Code for reproducibility.

**Type:** Practical | **Category:** Configuration as Code

## The Scenario

Your Jenkins environment has grown organically:

```
Current Problems:
- 3 Jenkins instances (dev, staging, prod) with different configurations
- No documentation of what plugins are installed or their versions
- Configuration changes made via UI are not tracked
- Disaster recovery takes days to manually reconfigure
- "It works on the dev Jenkins" is a common complaint
```

A recent incident where the production Jenkins was misconfigured caused a 4-hour outage. Leadership wants reproducible, version-controlled infrastructure.

## The Challenge

Implement Jenkins Configuration as Code (JCasC) to manage all Jenkins configuration declaratively, enabling version control, peer review, and automated deployment.


### Step 1: Export Current Configuration

```groovy
// Script to export current configuration (run in Script Console)


// Export current configuration to YAML
def configPath = "/tmp/jenkins-config-export.yaml"
ConfigurationAsCode.get().export(new FileOutputStream(configPath))
println "Configuration exported to: ${configPath}"
println new File(configPath).text
```

### Step 2: Create Comprehensive JCasC Configuration

```yaml
# jenkins.yaml - Main configuration file
jenkins:
  systemMessage: "Jenkins - Managed by Configuration as Code"
  numExecutors: 0  # No builds on master
  mode: EXCLUSIVE

  # Security configuration
  securityRealm:
    ldap:
      configurations:
        - server: "${LDAP_SERVER}"
          rootDN: "dc=company,dc=com"
          userSearchBase: "ou=users"
          userSearchFilter: "uid={0}"
          groupSearchBase: "ou=groups"
          managerDN: "${LDAP_MANAGER_DN}"
          managerPasswordSecret: "${LDAP_MANAGER_PASSWORD}"

  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions:
              - "Overall/Administer"
            entries:
              - group: "jenkins-admins"
          - name: "developer"
            permissions:
              - "Overall/Read"
              - "Job/Build"
              - "Job/Read"
              - "Job/Cancel"
            entries:
              - group: "developers"

  # Global properties
  globalNodeProperties:
    - envVars:
        env:
          - key: "COMPANY_REGISTRY"
            value: "registry.company.com"
          - key: "DEPLOY_ENV"
            value: "${DEPLOY_ENVIRONMENT:-development}"

  # Agent configuration
  nodes:
    - permanent:
        name: "static-agent-01"
        remoteFS: "/var/jenkins"
        numExecutors: 4
        launcher:
          ssh:
            host: "agent-01.company.com"
            credentialsId: "jenkins-ssh-key"
            sshHostKeyVerificationStrategy: "knownHostsFileKeyVerificationStrategy"
        labelString: "linux docker"

  clouds:
    - kubernetes:
        name: "kubernetes"
        serverUrl: "${K8S_SERVER_URL}"
        namespace: "jenkins"
        credentialsId: "k8s-service-account"
        jenkinsUrl: "http://jenkins:8080"
        jenkinsTunnel: "jenkins-agent:50000"
        containerCapStr: "50"
        templates:
          - name: "default"
            label: "kubernetes"
            containers:
              - name: "jnlp"
                image: "jenkins/inbound-agent:latest"
                resourceRequestMemory: "256Mi"
                resourceLimitMemory: "512Mi"

# Credentials configuration
credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: "github-credentials"
              username: "${GITHUB_USERNAME}"
              password: "${GITHUB_TOKEN}"
              description: "GitHub API credentials"
          - string:
              id: "slack-token"
              secret: "${SLACK_BOT_TOKEN}"
              description: "Slack notification token"
          - file:
              id: "kubeconfig-prod"
              fileName: "kubeconfig"
              secretBytes: "${base64:${KUBECONFIG_PROD}}"
              description: "Production Kubernetes config"
          - basicSSHUserPrivateKey:
              id: "jenkins-ssh-key"
              username: "jenkins"
              privateKeySource:
                directEntry:
                  privateKey: "${SSH_PRIVATE_KEY}"
              description: "SSH key for agent connections"

# Unclassified (plugin configurations)
unclassified:
  location:
    url: "${JENKINS_URL}"
    adminAddress: "jenkins-admin@company.com"

  gitHubPluginConfig:
    configs:
      - name: "GitHub"
        apiUrl: "https://api.github.com"
        credentialsId: "github-credentials"
        manageHooks: true

  slackNotifier:
    teamDomain: "company"
    tokenCredentialId: "slack-token"
    room: "#jenkins-notifications"

  globalLibraries:
    libraries:
      - name: "company-shared-library"
        retriever:
          modernSCM:
            scm:
              git:
                remote: "https://github.com/company/jenkins-shared-library.git"
                credentialsId: "github-credentials"
        defaultVersion: "main"
        implicit: true

  # Tool installations
  tool:
    maven:
      installations:
        - name: "maven-3.9"
          properties:
            - installSource:
                installers:
                  - maven:
                      id: "3.9.4"
    nodejs:
      installations:
        - name: "node-18"
          properties:
            - installSource:
                installers:
                  - nodeJSInstaller:
                      id: "18.17.1"
                      npmPackagesRefreshHours: 72
    jdk:
      installations:
        - name: "jdk-17"
          properties:
            - installSource:
                installers:
                  - adoptOpenJdkInstaller:
                      id: "jdk-17.0.7+7"
```

### Step 3: Manage Plugins Declaratively

```yaml
# plugins.yaml - Pin plugin versions
plugins:
  - artifactId: configuration-as-code
    version: "1714.v09593e830cfa"
  - artifactId: job-dsl
    version: "1.84"
  - artifactId: workflow-aggregator
    version: "596.v8c21c963d92d"
  - artifactId: git
    version: "5.2.0"
  - artifactId: github
    version: "1.37.1"
  - artifactId: kubernetes
    version: "3900.va_dce992317b_4"
  - artifactId: role-strategy
    version: "633.v836e5b_3e80a_5"
  - artifactId: credentials-binding
    version: "631.v861c6e55c903"
  - artifactId: slack
    version: "664.vc9a_90f8b_c24a_"
  - artifactId: blueocean
    version: "1.27.5"
```

```groovy
// Dockerfile for Jenkins with pinned plugins
FROM jenkins/jenkins:lts-jdk17

# Skip setup wizard
ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"

# Install plugins from plugins.txt
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

# Copy JCasC configuration
COPY jenkins.yaml /var/jenkins_home/casc_configs/
ENV CASC_JENKINS_CONFIG=/var/jenkins_home/casc_configs
```

### Step 4: Create GitOps Workflow

```yaml
# .github/workflows/jenkins-config.yaml
name: Deploy Jenkins Configuration

on:
  push:
    branches: [main]
    paths:
      - 'jenkins/**'
  pull_request:
    branches: [main]
    paths:
      - 'jenkins/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate YAML syntax
        run: |
          pip install yamllint
          yamllint jenkins/*.yaml

      - name: Validate JCasC schema
        run: |
          docker run --rm \
            -v $(pwd)/jenkins:/workspace \
            jenkins/jenkins:lts-jdk17 \
            java -jar /usr/share/jenkins/jenkins.war \
              --httpPort=-1 \
              --argumentsRealm.passwd.admin=admin \
              --argumentsRealm.roles.admin=admin \
              -Dcasc.jenkins.config=/workspace/jenkins.yaml \
              -Dcasc.validateOnStartup=true \
              || true

  deploy-staging:
    needs: validate
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Staging Jenkins
        run: |
          curl -X POST \
            -u "${{ secrets.JENKINS_USER }}:${{ secrets.JENKINS_TOKEN }}" \
            "${{ secrets.STAGING_JENKINS_URL }}/configuration-as-code/reload"

  deploy-production:
    needs: validate
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Deploy to Production Jenkins
        run: |
          curl -X POST \
            -u "${{ secrets.JENKINS_USER }}:${{ secrets.JENKINS_TOKEN }}" \
            "${{ secrets.PROD_JENKINS_URL }}/configuration-as-code/reload"
```

### Step 5: Handle Secrets Securely

```yaml
# Using environment variables (injected at runtime)
jenkins:
  systemMessage: "Environment: ${ENVIRONMENT}"

credentials:
  system:
    domainCredentials:
      - credentials:
          # Reference secrets from environment
          - string:
              id: "api-key"
              secret: "${API_KEY}"

          # Or from HashiCorp Vault
          - vaultStringCredentialBinding:
              id: "vault-secret"
              path: "secret/data/jenkins/api-key"
              vaultKey: "value"
```

```groovy
// Kubernetes ConfigMap and Secret for Jenkins
apiVersion: v1
kind: ConfigMap
metadata:
  name: jenkins-casc-config
data:
  jenkins.yaml: |
    jenkins:
      systemMessage: "Production Jenkins"
      # ... rest of config
---
apiVersion: v1
kind: Secret
metadata:
  name: jenkins-secrets
type: Opaque
data:
  GITHUB_TOKEN: <base64-encoded>
  SLACK_BOT_TOKEN: <base64-encoded>
  LDAP_MANAGER_PASSWORD: <base64-encoded>
```

### Step 6: Configuration Reload Pipeline

```groovy
// Jenkinsfile for configuration updates
pipeline {
    agent any

    triggers {
        githubPush()  // Trigger on config repo push
    }

    stages {
        stage('Checkout Config') {
            steps {
                git(
                    url: 'https://github.com/company/jenkins-config.git',
                    credentialsId: 'github-credentials',
                    branch: 'main'
                )
            }
        }

        stage('Validate Configuration') {
            steps {
                script {
                    // Validate YAML
                    sh 'yamllint jenkins.yaml'

                    // Dry run validation
                    def validation = httpRequest(
                        url: "${JENKINS_URL}/configuration-as-code/checkNewSource",
                        httpMode: 'POST',
                        authentication: 'jenkins-api-credentials',
                        contentType: 'APPLICATION_JSON',
                        requestBody: readFile('jenkins.yaml')
                    )

                    if (validation.status != 200) {
                        error "Configuration validation failed"
                    }
                }
            }
        }

        stage('Apply Configuration') {
            steps {
                script {
                    httpRequest(
                        url: "${JENKINS_URL}/configuration-as-code/reload",
                        httpMode: 'POST',
                        authentication: 'jenkins-api-credentials'
                    )
                }
            }
        }

        stage('Verify') {
            steps {
                script {
                    // Wait for Jenkins to reload
                    sleep(30)

                    // Verify Jenkins is healthy
                    def health = httpRequest(
                        url: "${JENKINS_URL}/api/json",
                        authentication: 'jenkins-api-credentials'
                    )

                    if (health.status != 200) {
                        error "Jenkins health check failed after config reload"
                    }
                }
            }
        }
    }

    post {
        failure {
            slackSend(
                channel: '#jenkins-alerts',
                color: 'danger',
                message: "JCasC update failed: ${env.BUILD_URL}"
            )
        }
        success {
            slackSend(
                channel: '#jenkins-ops',
                color: 'good',
                message: "JCasC configuration updated successfully"
            )
        }
    }
}
```


## JCasC Coverage Checklist

| Configuration Area | JCasC Section | Secrets Handling |
|--------------------|---------------|------------------|
| System settings | jenkins: | Environment vars |
| Security realm | jenkins.securityRealm | Vault/K8s secrets |
| Authorization | jenkins.authorizationStrategy | N/A |
| Credentials | credentials: | External secrets |
| Clouds (K8s/EC2) | jenkins.clouds | Service accounts |
| Tools | tool: | N/A |
| Plugins | Use plugins.txt | N/A |
| Global libraries | unclassified.globalLibraries | Git credentials |


---

### Quick Check

**What is the recommended way to handle secrets in Jenkins Configuration as Code?**

   A. Store them directly in the YAML file committed to Git
   B. Use base64 encoding in the configuration
-> C. **Use environment variable references that are injected at runtime from external secret managers**
   D. Create secrets manually in the UI and don't include them in JCasC

<details>
<summary>See Answer</summary>

JCasC supports variable substitution using ${VARIABLE_NAME} syntax. Secrets should be stored in external secret managers (HashiCorp Vault, Kubernetes Secrets, AWS Secrets Manager) and injected as environment variables at runtime. This keeps secrets out of version control while maintaining the benefits of configuration as code. Never commit secrets directly to Git, even if encoded.

</details>

---

### Question 11: Design a GitOps workflow with Jenkins for automated, auditable deployments to multiple environments.

**Type:** Architecture | **Category:** GitOps & Deployment

## The Scenario

Your organization struggles with deployment consistency:

```
Current Problems:
- Developers deploy manually via kubectl from laptops
- No audit trail of what was deployed when
- Staging doesn't match production configuration
- Rollbacks are manual and error-prone
- Environment drift causes "works on staging" issues
```

Compliance requires:
- All deployments must be traceable to a Git commit
- All production changes must be reviewed
- Ability to roll back to any previous version
- Full audit log of who deployed what

## The Challenge

Design and implement a GitOps workflow using Jenkins that provides automated, auditable, and repeatable deployments across development, staging, and production environments.


### Step 1: Design GitOps Repository Structure

```
# Separate repositories for separation of concerns

app-repo/                    # Application source code
├── src/
├── Dockerfile
├── Jenkinsfile              # CI pipeline (build, test, push image)
└── package.json

infra-repo/                  # Infrastructure/deployment configuration
├── base/                    # Base Kubernetes manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── kustomization.yaml
├── environments/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   ├── replicas-patch.yaml
│   │   └── resources-patch.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   ├── replicas-patch.yaml
│   │   └── resources-patch.yaml
│   └── production/
│       ├── kustomization.yaml
│       ├── replicas-patch.yaml
│       ├── hpa.yaml
│       └── pdb.yaml
├── Jenkinsfile              # CD pipeline (deploy to environments)
└── CODEOWNERS               # Require approvals for prod changes
```

### Step 2: CI Pipeline - Build and Update Manifests

```groovy
// app-repo/Jenkinsfile - CI Pipeline
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:24-cli
    command: ["sleep", "infinity"]
  - name: dind
    image: docker:24-dind
    securityContext:
      privileged: true
'''
        }
    }

    environment {
        REGISTRY = 'registry.company.com'
        APP_NAME = 'my-application'
        INFRA_REPO = 'github.com/company/infra-repo.git'
    }

    stages {
        stage('Build & Test') {
            steps {
                container('docker') {
                    sh '''
                        docker build -t ${REGISTRY}/${APP_NAME}:${GIT_COMMIT} .
                        docker run --rm ${REGISTRY}/${APP_NAME}:${GIT_COMMIT} npm test
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'registry-creds',
                        usernameVariable: 'REG_USER',
                        passwordVariable: 'REG_PASS'
                    )]) {
                        sh '''
                            echo $REG_PASS | docker login ${REGISTRY} -u $REG_USER --password-stdin
                            docker push ${REGISTRY}/${APP_NAME}:${GIT_COMMIT}

                            # Tag as latest for the branch
                            docker tag ${REGISTRY}/${APP_NAME}:${GIT_COMMIT} \
                                ${REGISTRY}/${APP_NAME}:${BRANCH_NAME}-latest
                            docker push ${REGISTRY}/${APP_NAME}:${BRANCH_NAME}-latest
                        '''
                    }
                }
            }
        }

        stage('Update Infra Repo') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    def targetEnv = (env.BRANCH_NAME == 'main') ? 'production' : 'staging'

                    withCredentials([usernamePassword(
                        credentialsId: 'github-credentials',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh """
                            # Clone infra repo
                            git clone https://${GIT_USER}:${GIT_TOKEN}@${INFRA_REPO} infra
                            cd infra

                            # Update image tag in kustomization
                            cd environments/${targetEnv}
                            kustomize edit set image ${REGISTRY}/${APP_NAME}:${GIT_COMMIT}

                            # Commit and push
                            git config user.email "jenkins@company.com"
                            git config user.name "Jenkins CI"
                            git add .
                            git commit -m "chore: Update ${APP_NAME} to ${GIT_COMMIT}

                            Source commit: ${GIT_COMMIT}
                            Source branch: ${BRANCH_NAME}
                            Jenkins build: ${BUILD_URL}
                            "

                            # For production, create PR instead of direct push
                            if [ "${targetEnv}" = "production" ]; then
                                git checkout -b deploy/${APP_NAME}-${GIT_COMMIT}
                                git push origin deploy/${APP_NAME}-${GIT_COMMIT}

                                # Create PR using GitHub CLI
                                gh pr create \
                                    --title "Deploy ${APP_NAME} to production (${GIT_COMMIT})" \
                                    --body "Automated deployment request\\n\\nSource: ${BUILD_URL}" \
                                    --reviewer platform-team
                            else
                                git push origin main
                            fi
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                def targetEnv = (env.BRANCH_NAME == 'main') ? 'production' : 'staging'
                slackSend(
                    channel: '#deployments',
                    color: 'good',
                    message: """
                        *Image Ready for Deployment*
                        App: ${APP_NAME}
                        Tag: ${GIT_COMMIT}
                        Environment: ${targetEnv}
                        ${targetEnv == 'production' ? '⚠️ PR created - awaiting approval' : '✅ Auto-deploying to staging'}
                    """
                )
            }
        }
    }
}
```

### Step 3: CD Pipeline - Deploy from Infra Repo

```groovy
// infra-repo/Jenkinsfile - CD Pipeline
pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["sleep", "infinity"]
'''
        }
    }

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'production'],
            description: 'Target environment'
        )
        booleanParam(
            name: 'DRY_RUN',
            defaultValue: false,
            description: 'Show what would be deployed without applying'
        )
    }

    stages {
        stage('Validate') {
            steps {
                container('kubectl') {
                    sh """
                        # Validate manifests
                        kustomize build environments/${params.ENVIRONMENT} | \
                            kubectl apply --dry-run=client -f -

                        echo "=== Resources to be deployed ==="
                        kustomize build environments/${params.ENVIRONMENT} | \
                            kubectl diff -f - || true
                    """
                }
            }
        }

        stage('Production Approval') {
            when {
                expression { params.ENVIRONMENT == 'production' && !params.DRY_RUN }
            }
            steps {
                script {
                    // Get list of changes
                    def changes = sh(
                        script: "git log --oneline HEAD~5..HEAD",
                        returnStdout: true
                    ).trim()

                    def approval = input(
                        message: "Deploy to Production?",
                        ok: 'Deploy',
                        submitter: 'release-managers,platform-team',
                        parameters: [
                            text(
                                name: 'APPROVAL_NOTE',
                                defaultValue: '',
                                description: 'Optional: Add deployment notes'
                            )
                        ]
                    )

                    env.APPROVER = currentBuild.getBuildCauses()[0]?.userId ?: 'unknown'
                    env.APPROVAL_NOTE = approval ?: ''
                }
            }
        }

        stage('Deploy') {
            when {
                expression { !params.DRY_RUN }
            }
            steps {
                container('kubectl') {
                    withCredentials([file(
                        credentialsId: "kubeconfig-${params.ENVIRONMENT}",
                        variable: 'KUBECONFIG'
                    )]) {
                        sh """
                            # Apply manifests
                            kustomize build environments/${params.ENVIRONMENT} | \
                                kubectl apply -f -

                            # Wait for rollout
                            kubectl rollout status deployment -n ${params.ENVIRONMENT} \
                                --timeout=300s
                        """
                    }
                }
            }
        }

        stage('Verify') {
            when {
                expression { !params.DRY_RUN }
            }
            steps {
                container('kubectl') {
                    withCredentials([file(
                        credentialsId: "kubeconfig-${params.ENVIRONMENT}",
                        variable: 'KUBECONFIG'
                    )]) {
                        sh """
                            # Health checks
                            kubectl get pods -n ${params.ENVIRONMENT}

                            # Run smoke tests
                            ENDPOINT=\$(kubectl get svc -n ${params.ENVIRONMENT} \
                                -o jsonpath='{.items[0].status.loadBalancer.ingress[0].hostname}')
                            curl -sf http://\$ENDPOINT/health || exit 1
                        """
                    }
                }
            }
        }

        stage('Record Deployment') {
            when {
                expression { !params.DRY_RUN }
            }
            steps {
                script {
                    // Create deployment record for audit
                    def deploymentRecord = [
                        timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ss'Z'"),
                        environment: params.ENVIRONMENT,
                        gitCommit: env.GIT_COMMIT,
                        gitBranch: env.GIT_BRANCH,
                        deployer: env.APPROVER ?: currentBuild.getBuildCauses()[0]?.userId,
                        buildUrl: env.BUILD_URL,
                        approvalNote: env.APPROVAL_NOTE ?: '',
                        status: 'success'
                    ]

                    writeJSON file: 'deployment-record.json', json: deploymentRecord

                    // Push to audit log
                    sh '''
                        # Send to audit system
                        curl -X POST https://audit.company.com/deployments \
                            -H "Content-Type: application/json" \
                            -d @deployment-record.json
                    '''

                    archiveArtifacts artifacts: 'deployment-record.json'
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#deployments',
                color: 'good',
                message: """
                    *Deployment Successful*
                    Environment: ${params.ENVIRONMENT}
                    Commit: ${env.GIT_COMMIT}
                    Deployed by: ${env.APPROVER ?: 'automated'}
                    Build: ${env.BUILD_URL}
                """
            )
        }
        failure {
            slackSend(
                channel: '#deployments',
                color: 'danger',
                message: """
                    *Deployment Failed*
                    Environment: ${params.ENVIRONMENT}
                    Build: ${env.BUILD_URL}
                    Please check logs and consider rollback
                """
            )
        }
    }
}
```

### Step 4: Implement Rollback Capability

```groovy
// rollback-pipeline.groovy
pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['staging', 'production'],
            description: 'Target environment'
        )
        string(
            name: 'TARGET_COMMIT',
            description: 'Git commit to rollback to (leave empty for previous)'
        )
    }

    stages {
        stage('Get Rollback Target') {
            steps {
                script {
                    if (params.TARGET_COMMIT?.trim()) {
                        env.ROLLBACK_COMMIT = params.TARGET_COMMIT
                    } else {
                        // Get previous deployment
                        env.ROLLBACK_COMMIT = sh(
                            script: "git log --skip=1 -1 --format=%H environments/${params.ENVIRONMENT}",
                            returnStdout: true
                        ).trim()
                    }

                    echo "Rolling back to commit: ${env.ROLLBACK_COMMIT}"
                }
            }
        }

        stage('Production Approval') {
            when {
                expression { params.ENVIRONMENT == 'production' }
            }
            steps {
                input(
                    message: "Confirm rollback to ${env.ROLLBACK_COMMIT}?",
                    ok: 'Rollback',
                    submitter: 'release-managers,platform-team'
                )
            }
        }

        stage('Execute Rollback') {
            steps {
                sh """
                    git checkout ${env.ROLLBACK_COMMIT} -- environments/${params.ENVIRONMENT}
                    git commit -m "Rollback ${params.ENVIRONMENT} to ${env.ROLLBACK_COMMIT}"
                    git push origin main
                """

                // Trigger deployment pipeline
                build(
                    job: 'infra-deploy',
                    parameters: [
                        string(name: 'ENVIRONMENT', value: params.ENVIRONMENT)
                    ],
                    wait: true
                )
            }
        }
    }
}
```

### Step 5: Environment Promotion Flow

```groovy
// promote-pipeline.groovy
pipeline {
    agent any

    parameters {
        choice(
            name: 'SOURCE_ENV',
            choices: ['dev', 'staging'],
            description: 'Source environment'
        )
        choice(
            name: 'TARGET_ENV',
            choices: ['staging', 'production'],
            description: 'Target environment'
        )
    }

    stages {
        stage('Validate Promotion Path') {
            steps {
                script {
                    def validPaths = [
                        'dev': ['staging'],
                        'staging': ['production']
                    ]

                    if (!validPaths[params.SOURCE_ENV].contains(params.TARGET_ENV)) {
                        error "Invalid promotion path: ${params.SOURCE_ENV} -> ${params.TARGET_ENV}"
                    }
                }
            }
        }

        stage('Get Current Version') {
            steps {
                script {
                    env.SOURCE_IMAGE = sh(
                        script: """
                            kustomize build environments/${params.SOURCE_ENV} | \
                            grep 'image:' | head -1 | awk '{print \$2}'
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Promoting image: ${env.SOURCE_IMAGE}"
                }
            }
        }

        stage('Update Target Environment') {
            steps {
                sh """
                    cd environments/${params.TARGET_ENV}
                    kustomize edit set image ${env.SOURCE_IMAGE}

                    git add .
                    git commit -m "Promote ${params.SOURCE_ENV} -> ${params.TARGET_ENV}

                    Image: ${env.SOURCE_IMAGE}
                    "
                    git push origin main
                """
            }
        }
    }
}
```


## GitOps Benefits Achieved

| Requirement | Implementation | Verification |
|-------------|----------------|--------------|
| Traceable deployments | All changes via Git | Git log shows full history |
| Reviewed changes | PRs for production | CODEOWNERS enforcement |
| Easy rollback | Git revert workflow | Rollback pipeline |
| Environment parity | Kustomize overlays | Diff between envs |
| Audit logging | Deployment records | Searchable audit trail |

<InterviewQuiz
  question="Why do GitOps workflows typically use a separate repository for infrastructure/deployment configuration instead of keeping it with the application code?"
  options={[
    "To reduce the size of the application repository",
    "Because Kubernetes doesn't support configuration in app repos",
    "To enable independent release cycles, cleaner audit trails, and prevent application developers from accidentally modifying production config",
    "To make CI/CD faster"
  ]}
  correctAnswer={2}
  explanation="Separating infrastructure configuration into its own repo provides several benefits: 1) Different access controls - app devs can't modify prod config without review, 2) Cleaner audit trail - deployment history is separate from app development, 3) Independent release cycles - you can change config without rebuilding the app, 4) Multiple apps can share the same deployment patterns. This separation of concerns is a core principle of GitOps."
/>

---

### Question 12: Jenkins server crashed and we lost all job configurations. Implement a disaster recovery strategy.

**Type:** Practical | **Category:** Backup & Recovery

## The Scenario

Monday morning: Jenkins is down. The disk failed overnight.

```
Current Impact:
- Jenkins master: OFFLINE (disk failure)
- Jobs configured: 500+ (all lost)
- Credentials: 150 (need to be recreated)
- Build history: 2 years (gone)
- Pipeline libraries: Custom shared library (gone)
- Estimated recovery: Unknown (no backup tested)
```

Developers can't deploy. The CEO is asking when services will be restored. You need to recover AND ensure this never happens again.

## The Challenge

Implement a comprehensive disaster recovery strategy that includes automated backups, quick restoration procedures, and infrastructure that can survive failures.


> **Common Mistake:** A junior engineer might just add a cron job to copy JENKINS_HOME to S3 without testing restores, skip credentials backup due to security concerns, or backup only on-demand when someone remembers. These approaches leave you unable to restore, miss critical data, and create false confidence.

> **Senior Engineer Approach:** A senior engineer implements automated, tested backups using multiple strategies (JCasC for config, encrypted credential backups, build history archival), creates runbooks for restoration, regularly tests recovery procedures, and optionally implements high availability to prevent the scenario entirely.

### Step 1: Define Recovery Objectives

```yaml
# recovery-objectives.yaml
disaster_recovery:
  RTO: 30 minutes      # Recovery Time Objective
  RPO: 1 hour          # Recovery Point Objective

  what_to_backup:
    critical:          # Must restore within RTO
      - job_configurations
      - credentials
      - plugin_list
      - global_configuration
      - shared_libraries
    important:         # Restore within 4 hours
      - build_history
      - workspace_caches
    nice_to_have:      # Restore within 24 hours
      - archived_artifacts
      - old_build_logs
```

### Step 2: Implement Configuration as Code Backup

```yaml
# Store Jenkins configuration in Git (primary backup)
# jenkins-config-repo/jenkins.yaml
jenkins:
  systemMessage: "Jenkins - Disaster Recovery Enabled"
  numExecutors: 0
  mode: EXCLUSIVE

  securityRealm:
    ldap:
      configurations:
        - server: "${LDAP_SERVER}"
          rootDN: "dc=company,dc=com"

  authorizationStrategy:
    roleBased:
      roles:
        global:
          - name: "admin"
            permissions:
              - "Overall/Administer"
            entries:
              - group: "jenkins-admins"

  clouds:
    - kubernetes:
        name: "kubernetes"
        serverUrl: "${K8S_SERVER_URL}"
        # ... full cloud config

credentials:
  system:
    domainCredentials:
      - credentials:
          # Secrets from external vault - never in Git
          - usernamePassword:
              id: "github-credentials"
              username: "${GITHUB_USERNAME}"
              password: "${GITHUB_TOKEN}"

unclassified:
  globalLibraries:
    libraries:
      - name: "company-shared-library"
        retriever:
          modernSCM:
            scm:
              git:
                remote: "https://github.com/company/jenkins-shared-library.git"
        defaultVersion: "main"
```

### Step 3: Automated Backup Pipeline

```groovy
// backup-pipeline.groovy
pipeline {
    agent { label 'backup-agent' }

    triggers {
        cron('0 */4 * * *')  // Every 4 hours
    }

    environment {
        BACKUP_BUCKET = 's3://jenkins-backups-company'
        JENKINS_HOME = '/var/jenkins_home'
        TIMESTAMP = sh(script: 'date +%Y%m%d-%H%M%S', returnStdout: true).trim()
    }

    stages {
        stage('Prepare Backup') {
            steps {
                sh '''
                    # Create backup directory
                    mkdir -p /tmp/jenkins-backup-${TIMESTAMP}
                    cd /tmp/jenkins-backup-${TIMESTAMP}

                    # Export configuration via JCasC
                    curl -s -u ${JENKINS_USER}:${JENKINS_TOKEN} \
                        ${JENKINS_URL}/configuration-as-code/export \
                        > jenkins-config.yaml

                    # List installed plugins with versions
                    curl -s -u ${JENKINS_USER}:${JENKINS_TOKEN} \
                        "${JENKINS_URL}/pluginManager/api/json?depth=1" | \
                        jq -r '.plugins[] | "\\(.shortName):\\(.version)"' | \
                        sort > plugins.txt
                '''
            }
        }

        stage('Backup Job Configurations') {
            steps {
                sh '''
                    cd /tmp/jenkins-backup-${TIMESTAMP}

                    # Backup all job configs (without build history)
                    mkdir -p jobs
                    find ${JENKINS_HOME}/jobs -name "config.xml" -type f | while read config; do
                        # Get relative path
                        relpath=$(echo "$config" | sed "s|${JENKINS_HOME}/jobs/||")
                        mkdir -p "jobs/$(dirname "$relpath")"
                        cp "$config" "jobs/$relpath"
                    done

                    # Backup nodes configuration
                    mkdir -p nodes
                    cp -r ${JENKINS_HOME}/nodes/* nodes/ 2>/dev/null || true

                    # Backup views
                    mkdir -p views
                    find ${JENKINS_HOME} -maxdepth 1 -name "*View*" -exec cp {} views/ \\;
                '''
            }
        }

        stage('Backup Credentials (Encrypted)') {
            steps {
                withCredentials([string(credentialsId: 'backup-encryption-key', variable: 'ENCRYPTION_KEY')]) {
                    sh '''
                        cd /tmp/jenkins-backup-${TIMESTAMP}

                        # Backup secrets directory (encrypted)
                        tar -czf - ${JENKINS_HOME}/secrets | \
                            openssl enc -aes-256-cbc -salt -pbkdf2 \
                            -pass pass:${ENCRYPTION_KEY} \
                            > secrets.tar.gz.enc

                        # Backup credentials.xml (encrypted)
                        openssl enc -aes-256-cbc -salt -pbkdf2 \
                            -pass pass:${ENCRYPTION_KEY} \
                            -in ${JENKINS_HOME}/credentials.xml \
                            -out credentials.xml.enc
                    '''
                }
            }
        }

        stage('Backup Build History') {
            when {
                // Only full backup on weekends
                expression { return new Date().format('u') in ['6', '7'] }
            }
            steps {
                sh '''
                    cd /tmp/jenkins-backup-${TIMESTAMP}

                    # Backup last 10 builds per job (excluding workspaces)
                    mkdir -p build-history
                    find ${JENKINS_HOME}/jobs -type d -name "builds" | while read builds_dir; do
                        job_name=$(echo "$builds_dir" | sed "s|${JENKINS_HOME}/jobs/||" | sed 's|/builds||')
                        mkdir -p "build-history/$job_name"

                        # Copy last 10 builds
                        ls -1t "$builds_dir" | head -10 | while read build; do
                            if [ -d "$builds_dir/$build" ]; then
                                cp -r "$builds_dir/$build" "build-history/$job_name/" 2>/dev/null || true
                            fi
                        done
                    done
                '''
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-backup-credentials',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh '''
                        cd /tmp

                        # Create compressed archive
                        tar -czf jenkins-backup-${TIMESTAMP}.tar.gz jenkins-backup-${TIMESTAMP}

                        # Upload to S3
                        aws s3 cp jenkins-backup-${TIMESTAMP}.tar.gz \
                            ${BACKUP_BUCKET}/daily/${TIMESTAMP}/

                        # Keep last 30 daily backups
                        aws s3 ls ${BACKUP_BUCKET}/daily/ | sort | head -n -30 | \
                            awk '{print $2}' | while read old_backup; do
                                aws s3 rm ${BACKUP_BUCKET}/daily/$old_backup --recursive
                            done

                        # Cleanup local
                        rm -rf jenkins-backup-${TIMESTAMP} jenkins-backup-${TIMESTAMP}.tar.gz
                    '''
                }
            }
        }

        stage('Verify Backup') {
            steps {
                script {
                    // Download and verify backup integrity
                    sh '''
                        cd /tmp
                        aws s3 cp ${BACKUP_BUCKET}/daily/${TIMESTAMP}/jenkins-backup-${TIMESTAMP}.tar.gz .
                        tar -tzf jenkins-backup-${TIMESTAMP}.tar.gz > /dev/null
                        echo "Backup verification: SUCCESS"
                        rm jenkins-backup-${TIMESTAMP}.tar.gz
                    '''
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#jenkins-ops',
                color: 'good',
                message: "Jenkins backup completed: ${TIMESTAMP}"
            )
        }
        failure {
            slackSend(
                channel: '#jenkins-alerts',
                color: 'danger',
                message: "Jenkins backup FAILED! Immediate attention required."
            )
        }
    }
}
```

### Step 4: Disaster Recovery Runbook

```groovy
// restore-jenkins.groovy - Run from recovery environment
pipeline {
    agent any

    parameters {
        string(
            name: 'BACKUP_TIMESTAMP',
            description: 'Backup timestamp to restore (e.g., 20240115-120000)'
        )
        booleanParam(
            name: 'RESTORE_CREDENTIALS',
            defaultValue: true,
            description: 'Restore encrypted credentials'
        )
        booleanParam(
            name: 'RESTORE_BUILD_HISTORY',
            defaultValue: false,
            description: 'Restore build history (takes longer)'
        )
    }

    environment {
        BACKUP_BUCKET = 's3://jenkins-backups-company'
        NEW_JENKINS_HOME = '/var/jenkins_home'
    }

    stages {
        stage('Download Backup') {
            steps {
                sh '''
                    mkdir -p /tmp/jenkins-restore
                    cd /tmp/jenkins-restore

                    aws s3 cp ${BACKUP_BUCKET}/daily/${BACKUP_TIMESTAMP}/jenkins-backup-${BACKUP_TIMESTAMP}.tar.gz .
                    tar -xzf jenkins-backup-${BACKUP_TIMESTAMP}.tar.gz
                '''
            }
        }

        stage('Stop Jenkins') {
            steps {
                sh '''
                    # Stop Jenkins gracefully
                    curl -X POST -u ${JENKINS_USER}:${JENKINS_TOKEN} \
                        "${JENKINS_URL}/safeExit" || true

                    # Wait for shutdown
                    sleep 30
                '''
            }
        }

        stage('Restore Configuration') {
            steps {
                sh '''
                    cd /tmp/jenkins-restore/jenkins-backup-${BACKUP_TIMESTAMP}

                    # Restore JCasC configuration
                    cp jenkins-config.yaml ${NEW_JENKINS_HOME}/casc_configs/

                    # Restore job configurations
                    cp -r jobs/* ${NEW_JENKINS_HOME}/jobs/

                    # Restore nodes
                    cp -r nodes/* ${NEW_JENKINS_HOME}/nodes/ 2>/dev/null || true
                '''
            }
        }

        stage('Restore Credentials') {
            when {
                expression { params.RESTORE_CREDENTIALS }
            }
            steps {
                withCredentials([string(credentialsId: 'backup-encryption-key', variable: 'ENCRYPTION_KEY')]) {
                    sh '''
                        cd /tmp/jenkins-restore/jenkins-backup-${BACKUP_TIMESTAMP}

                        # Restore secrets
                        openssl enc -aes-256-cbc -d -pbkdf2 \
                            -pass pass:${ENCRYPTION_KEY} \
                            -in secrets.tar.gz.enc | tar -xzf - -C ${NEW_JENKINS_HOME}/

                        # Restore credentials.xml
                        openssl enc -aes-256-cbc -d -pbkdf2 \
                            -pass pass:${ENCRYPTION_KEY} \
                            -in credentials.xml.enc \
                            -out ${NEW_JENKINS_HOME}/credentials.xml
                    '''
                }
            }
        }

        stage('Install Plugins') {
            steps {
                sh '''
                    cd /tmp/jenkins-restore/jenkins-backup-${BACKUP_TIMESTAMP}

                    # Install plugins from list
                    jenkins-plugin-cli --plugin-file plugins.txt
                '''
            }
        }

        stage('Start Jenkins') {
            steps {
                sh '''
                    # Start Jenkins
                    systemctl start jenkins

                    # Wait for startup
                    timeout 300 bash -c 'until curl -s ${JENKINS_URL}/login; do sleep 5; done'

                    echo "Jenkins is up and running"
                '''
            }
        }

        stage('Verify Restoration') {
            steps {
                script {
                    // Verify jobs are present
                    def jobCount = sh(
                        script: """
                            curl -s -u ${JENKINS_USER}:${JENKINS_TOKEN} \
                                "${JENKINS_URL}/api/json?tree=jobs[name]" | \
                                jq '.jobs | length'
                        """,
                        returnStdout: true
                    ).trim().toInteger()

                    echo "Restored ${jobCount} jobs"

                    if (jobCount < 100) {  // Expected minimum
                        error "Job count lower than expected!"
                    }

                    // Test a known credential
                    def credTest = sh(
                        script: """
                            curl -s -u ${JENKINS_USER}:${JENKINS_TOKEN} \
                                "${JENKINS_URL}/credentials/store/system/domain/_/credential/github-credentials/api/json"
                        """,
                        returnStatus: true
                    )

                    if (credTest != 0) {
                        error "Credential restoration verification failed!"
                    }

                    echo "Restoration verified successfully"
                }
            }
        }
    }

    post {
        success {
            slackSend(
                channel: '#jenkins-alerts',
                color: 'good',
                message: """
                    *Jenkins Restored Successfully*
                    Backup: ${params.BACKUP_TIMESTAMP}
                    Time: ${currentBuild.durationString}
                """
            )
        }
        always {
            sh 'rm -rf /tmp/jenkins-restore'
        }
    }
}
```

### Step 5: High Availability Setup (Prevention)

```yaml
# kubernetes/jenkins-ha.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: jenkins
  namespace: jenkins
spec:
  serviceName: jenkins
  replicas: 1  # Active-passive with persistent storage
  selector:
    matchLabels:
      app: jenkins
  template:
    spec:
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts-jdk17
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
          resources:
            requests:
              memory: "4Gi"
              cpu: "2"
            limits:
              memory: "8Gi"
              cpu: "4"
  volumeClaimTemplates:
    - metadata:
        name: jenkins-home
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: gp3  # High IOPS SSD
        resources:
          requests:
            storage: 100Gi
---
# Use EBS snapshots for point-in-time recovery
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: jenkins-snapshot-class
driver: ebs.csi.aws.com
deletionPolicy: Retain
parameters:
  tagSpecification_1: "Environment=production"
---
# Automated daily snapshots
apiVersion: batch/v1
kind: CronJob
metadata:
  name: jenkins-volume-snapshot
  namespace: jenkins
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: snapshot-creator
              image: bitnami/kubectl
              command:
                - /bin/sh
                - -c
                - |
                  kubectl apply -f - <<EOF
                  apiVersion: snapshot.storage.k8s.io/v1
                  kind: VolumeSnapshot
                  metadata:
                    name: jenkins-home-$(date +%Y%m%d)
                    namespace: jenkins
                  spec:
                    volumeSnapshotClassName: jenkins-snapshot-class
                    source:
                      persistentVolumeClaimName: jenkins-home-jenkins-0
                  EOF
          restartPolicy: OnFailure
```

### Step 6: Regular Recovery Testing

```groovy
// test-disaster-recovery.groovy - Run monthly
pipeline {
    agent any

    triggers {
        cron('0 3 1 * *')  // First of every month at 3 AM
    }

    stages {
        stage('Create Test Environment') {
            steps {
                sh '''
                    # Spin up temporary Jenkins instance
                    docker run -d --name jenkins-dr-test \
                        -p 9090:8080 \
                        jenkins/jenkins:lts-jdk17
                '''
            }
        }

        stage('Restore to Test Environment') {
            steps {
                // Use the restore pipeline on test instance
                build(
                    job: 'restore-jenkins',
                    parameters: [
                        string(name: 'BACKUP_TIMESTAMP', value: 'latest'),
                        booleanParam(name: 'TARGET_INSTANCE', value: 'test')
                    ]
                )
            }
        }

        stage('Validate Restoration') {
            steps {
                script {
                    // Run comprehensive tests
                    def tests = [
                        'Jobs present': 'curl -s localhost:9090/api/json | jq ".jobs | length"',
                        'Plugins loaded': 'curl -s localhost:9090/pluginManager/api/json | jq ".plugins | length"',
                        'Credentials accessible': 'curl -s localhost:9090/credentials/api/json'
                    ]

                    tests.each { name, command ->
                        def result = sh(script: command, returnStatus: true)
                        if (result != 0) {
                            error "DR Test Failed: ${name}"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker rm -f jenkins-dr-test || true'
        }
        success {
            emailext(
                to: 'ops-team@company.com',
                subject: 'Jenkins DR Test: PASSED',
                body: 'Monthly disaster recovery test completed successfully.'
            )
        }
        failure {
            emailext(
                to: 'ops-team@company.com',
                subject: 'Jenkins DR Test: FAILED',
                body: 'Monthly disaster recovery test FAILED. Immediate attention required.'
            )
        }
    }
}
```


## Disaster Recovery Checklist

| Component | Backup Method | Frequency | Restore Time |
|-----------|---------------|-----------|--------------|
| Job configs | Git + S3 | Every 4 hours | 5 minutes |
| Credentials | Encrypted S3 | Every 4 hours | 2 minutes |
| Plugins list | Git | On change | 10 minutes |
| JCasC config | Git | On change | 2 minutes |
| Build history | S3 | Weekly | 30 minutes |
| Shared libraries | Git | On change | 1 minute |

---

## Practice with Real Infrastructure

Reading questions is good. **Deploying real projects is better.**

These interview questions teach you the concepts. To truly master them, you need hands-on practice with real cloud infrastructure.

**[Try DeployU Free](https://deployu.ai?ref=github)** - Deploy Docker containers, Kubernetes clusters, and cloud applications on real AWS/Azure/GCP infrastructure. No credit card required.

---

*Found this helpful? [Star the repo](https://github.com/gsraju27/ai-cloud-devops-roadmap) to support the project!*
