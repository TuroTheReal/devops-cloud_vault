# Taskfile - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, taskfile, automation, task-runner]
created: 2025-12-23
updated: 2025-12-23
```

**Related**: [[makefile]], [[automation-tools]]

---

## 📚 Table of Contents

1. [Quick Start](#-quick-start)
2. [Basic Syntax](#-basic-syntax)
3. [Variables](#-variables)
4. [Advanced Task Features](#-advanced-task-features)
5. [Real-World Example from Glasck](#-real-world-example-from-glasck)
6. [Common Patterns](#-common-patterns)
7. [Useful Commands](#️-useful-commands)
8. [Best Practices](#-best-practices)
9. [Common Patterns from Glasck](#-common-patterns-from-glasck)

---

## 🚀 Quick Start

```yaml
# Taskfile.yml
version: '3'

vars:
  GREETING: Hello, World!

tasks:
  hello:
    desc: Print a greeting
    cmds:
      - echo "{{.GREETING}}"

  build:
    desc: Build the project
    cmds:
      - go build -o app main.go
```

```bash
# Install
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b /usr/local/bin

# Run task
task hello

# List available tasks
task --list

# Show task summary
task --list-all
```

---

## 📝 Basic Syntax

### Simple Task
```yaml
tasks:
  build:
    desc: Build the application
    cmds:
      - go build -o app
```

### Multi-Command Task
```yaml
tasks:
  deploy:
    desc: Deploy the application
    cmds:
      - cmd: echo "🚀 Starting deployment..."
      - docker build -t myapp:latest .
      - docker push myapp:latest
      - cmd: echo "✅ Deployment complete!"
```

### Task Dependencies
```yaml
tasks:
  build:
    cmds:
      - go build -o app

  test:
    deps: [build]  # Run 'build' first
    cmds:
      - go test ./...

  deploy:
    deps: [test]  # Run 'test' (which runs 'build') first
    cmds:
      - ./deploy.sh
```

---

## 🔧 Variables

### Global Variables
```yaml
vars:
  API_VERSION: "1.2.0"
  FRONT_VERSION: "2.1.0"
  PROJECT_NAME: myproject

tasks:
  version:
    cmds:
      - echo "API: {{.API_VERSION}}"
      - echo "Frontend: {{.FRONT_VERSION}}"
```

### Environment Variables
```yaml
tasks:
  deploy:
    env:
      NODE_ENV: production
      DEBUG: "false"
    cmds:
      - npm run deploy
```

### Dynamic Variables (from commands)
```yaml
tasks:
  version:
    vars:
      GIT_COMMIT:
        sh: git rev-parse --short HEAD
    cmds:
      - echo "Commit: {{.GIT_COMMIT}}"
```

### User Input (CLI variables)
```yaml
tasks:
  deploy:
    cmds:
      - echo "Deploying {{.ENV}}"

# Usage: task deploy ENV=production
```

---

## 🔄 Advanced Task Features

### Silent Mode
```yaml
tasks:
  build:
    silent: true  # Don't echo commands
    cmds:
      - echo "Building..."
      - go build
```

### Conditional Execution
```yaml
tasks:
  build:
    cmds:
      - task: build-linux
        if: eq OS "linux"
      - task: build-windows
        if: eq OS "windows"
```

### Platforms (from Glasck)
```yaml
tasks:
  setup:
    desc: Setup on any platform
    cmds:
      - |
        {{if eq OS "windows"}}
          powershell -File ./setup.ps1
        {{else}}
          bash ./setup.sh
        {{end}}
```

### File Watching
```yaml
tasks:
  dev:
    desc: Development mode with auto-reload
    watch: true
    sources:
      - "**/*.go"
    cmds:
      - go run main.go
```

### User Prompts
```yaml
tasks:
  clean:
    desc: Clean all data
    prompt: "⚠️  Are you sure you want to delete everything?"
    cmds:
      - rm -rf data/
      - docker system prune -af
```

---

## 📦 Real-World Example from Glasck

```yaml
version: '3'

vars:
  # Repository configuration
  FRONT_REPO: git@github.com:user/frontend.git
  API_REPO: git@github.com:user/api.git

  # Service mapping
  SERVICES: |
    frontend:{{.FRONT_REPO}}
    api:{{.API_REPO}}

tasks:
  # Setup - Clone repositories
  setup-linux:
    desc: Clone repos into service/[name]/src (Linux/Mac)
    cmds:
      - |
        echo "🔄 Cloning repositories..."
        services="{{.SERVICES}}"
        while IFS= read -r line; do
          if [ -n "$line" ]; then
            service=$(echo "$line" | cut -d':' -f1)
            repo=$(echo "$line" | cut -d':' -f2-)
            echo "⬇️  Processing $service..."

            servicePath="service/$service/src"

            if [ ! -d "$servicePath/.git" ]; then
              echo "🚀 Cloning $service..."
              git clone "$repo" "$servicePath"
            else
              echo "🔄 Updating $service..."
              cd "$servicePath" && git pull && cd -
            fi
          fi
        done <<< "$services"
        echo "✅ Setup complete!"
    silent: true

  # Deploy to production
  prod-deploy:
    desc: 🏭 Deploy to production
    cmds:
      - cmd: echo "🏭 Starting production deployment..."
      - task: vps-deploy
      - docker-compose -f docker/docker-compose.prod.yml up -d --build
      - task: safe-clean
      - cmd: echo "✅ Production deployment complete!"
    silent: true

  # Swarm deployment
  swarm-build:
    desc: 🛠️  Build Docker images for Swarm
    cmds:
      - |
        API_VERSION=$(cat service/api/VERSION)
        FRONT_VERSION=$(cat service/frontend/VERSION)

        echo "🔨 Building API: ${API_VERSION}"
        echo "🔨 Building Frontend: ${FRONT_VERSION}"

        docker compose -f docker/stack.dev.yml build

        docker tag myapp-api:latest myapp-api:${API_VERSION}
        docker tag myapp-frontend:latest myapp-frontend:${FRONT_VERSION}

        echo "✅ Images built"
    silent: true

  swarm-deploy:
    desc: 🏭 Deploy to Docker Swarm
    cmds:
      - |
        set -a
        source docker/dev.env
        set +a

        export API_VERSION=$(cat service/api/VERSION)
        export FRONT_VERSION=$(cat service/frontend/VERSION)

        echo "📦 Deploying infrastructure..."
        docker stack deploy -c docker/stack.vps.yml myapp

        echo "⏳ Waiting for Redis..."
        until docker service ls --filter name=myapp_redis --format '{{.Replicas}}' | grep -q "1/1"; do
          printf "."
          sleep 2
        done
        echo " ✅"

        echo "🚀 Deploying services..."
        docker stack deploy -c docker/stack.dev.yml myapp
    silent: true

  swarm:
    desc: 🏭 Build + Deploy to Swarm
    deps: [swarm-build]
    cmds:
      - task: swarm-deploy
      - docker image prune -f
      - cmd: echo "✅ Deployment complete!"

  # Utilities
  status:
    desc: 📊 Show Docker status
    cmds:
      - cmd: echo "📊 Container Status:"
      - docker ps --format "table {{.Names}}\t{{.Status}}"
      - cmd: echo ""
      - cmd: echo "💾 Disk Usage:"
      - docker system df
    silent: true

  logs:
    desc: 📄 View service logs
    cmds:
      - |
        read -p "📄 Service name: " service_name
        if [ -z "$service_name" ]; then
          echo "❌ Service name required!"
          exit 1
        fi
        docker service logs myapp_$service_name --tail 50 --follow
    silent: true

  clean:
    desc: ⚠️  Clean all Docker resources
    prompt: "⚠️  Delete EVERYTHING?"
    cmds:
      - docker stack rm myapp || true
      - docker system prune -af --volumes
      - cmd: echo "✅ Cleanup complete"
    silent: true
```

---

## 🔍 Common Patterns

### Loop Over Files
```yaml
tasks:
  process-files:
    cmds:
      - |
        for file in *.txt; do
          echo "Processing $file"
          cat "$file"
        done
```

### Conditional File Check
```yaml
tasks:
  build:
    cmds:
      - |
        if [ -f "go.mod" ]; then
          echo "Building Go project"
          go build
        elif [ -f "package.json" ]; then
          echo "Building Node project"
          npm run build
        fi
```

### Parallel Task Execution
```yaml
tasks:
  test-all:
    deps:
      - unit-tests
      - integration-tests
      - e2e-tests
    # All deps run in parallel
```

### Sequential Task Execution
```yaml
tasks:
  deploy:
    cmds:
      - task: build
      - task: test
      - task: push
    # Tasks run sequentially
```

---

## 🛠️ Useful Commands

```bash
# Run task
task build

# Run with variable
task deploy ENV=production

# Run specific task from subdirectory
task -d /path/to/project build

# List all tasks
task --list
task -l

# Show verbose output
task --verbose build
task -v build

# Dry run (show commands without executing)
task --dry build

# Force run (ignore up-to-date check)
task --force build

# Run task in parallel
task --parallel test-unit test-integration

# Watch for file changes
task --watch dev
```

---

## 📚 Best Practices

1. **Descriptive Names**: Use `desc:` for every task
2. **Silent Mode**: Use `silent: true` for cleaner output
3. **Error Handling**: Use `set -e` in shell scripts
4. **Variables**: Centralize configuration in `vars:`
5. **Dependencies**: Use `deps:` instead of calling tasks within `cmds:`
6. **Platform Support**: Use `{{if eq OS "windows"}}` for cross-platform
7. **Prompts**: Add `prompt:` for destructive operations
8. **Emojis**: Use emojis in `echo` for visual feedback
9. **Exit Codes**: Check `$?` after critical commands
10. **Documentation**: Add comments for complex tasks

---

## 🚨 Common Patterns from Glasck

### Service Loop Pattern
```yaml
cmds:
  - |
    services="{{.SERVICES}}"
    while IFS= read -r line; do
      if [ -n "$line" ]; then
        service=$(echo "$line" | cut -d':' -f1)
        repo=$(echo "$line" | cut -d':' -f2-)
        echo "Processing $service from $repo"
      fi
    done <<< "$services"
```

### Version Management
```yaml
cmds:
  - |
    API_VERSION=$(cat service/api/VERSION)
    echo "Deploying version: ${API_VERSION}"
    docker tag myapp:latest myapp:${API_VERSION}
```

### Wait for Service Ready
```yaml
cmds:
  - |
    echo "⏳ Waiting for service..."
    until docker service ls --filter name=myapp_redis --format '{{.Replicas}}' | grep -q "1/1"; do
      printf "."
      sleep 2
    done
    echo " ✅"
```

### Interactive Input
```yaml
cmds:
  - |
    read -p "Enter service name: " service_name
    if [ -z "$service_name" ]; then
      echo "❌ Service name required!"
      exit 1
    fi
    docker logs $service_name
```

---

**Last updated**: 2025-12-23
**Source**: Extracted from Glasck Taskfile.yml
